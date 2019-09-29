---
layout:     post
title:      2019-09-20-记一次技术含量很低的iOS13适配
subtitle:   iOS日常积累
date:       2019-09-20
author:     二狗子
header-img: img/home-bg-o.jpg
catalog: true
tags:
    - iOS日常积累
    - OC
---
# 记一次技术含量很低的iOS13适配

## 前言

事情的起因是这样的，目前维护的项目是从创建至今已经5年多了，其中的很多图片资源都是适用于4英寸，而且布局全部使用frame。

这样就会出现一个问题，老的图片文件是不能直接用于现在5英寸以上的屏幕的，而且以前的frame对于现在的机器来说也是过小的。

在iphoneX发布的时候，之前维护代码的同事针对iphoneX进行了适配，但是判断机型的方式是这样的
```
([UIScreen instancesRespondToSelector:@selector(currentMode)] ? CGSizeEqualToSize(CGSizeMake(1125, 2436), [[UIScreen mainScreen] currentMode].size) : NO)
```
这个用在了判断刘海屏的地方，在当时是没有问题的，因为当时只有iphoneX所以没有问题。但是后来出现了iphoneXR这个异类，使用这个方法是判断不住的，因为一旦XR进入放大模式这个根据屏幕size的判断就会失效，如果多增加判断条件显得就会很臃肿，不方便后续维护。
我接手的时候正好是XR发布，页面很多地方出现问题。于是我想到了另一种判断方式。

```
	size_t size;
    sysctlbyname("hw.machine", NULL, &size, NULL, 0);
    char *machine = malloc(size);
    sysctlbyname("hw.machine", machine, &size, NULL, 0);
    NSString *devicePlateform = [NSString stringWithCString:machine encoding:NSUTF8StringEncoding];
    free(machine);
```

这个方法会返回类似于`iPhone1,1`的机型号码，所以XR是否放大模式都可以判断了。
当时一顿窃喜，我就在需要适配的地方又加了一条判断，最终判断逻辑变成了类似这样。

```
IS_iPhone_X || [DeviceUtils deviceType] == EDeviceIPhoneXR || IS_iPhone_XSM
```

关于`IS_iPhone_XSM`的判断与iphoneX是一样的。

```
([UIScreen instancesRespondToSelector:@selector(currentMode)] ? CGSizeEqualToSize(CGSizeMake(1242, 2688), [[UIScreen mainScreen] currentMode].size) : NO)

```

至此我们通过屏幕尺寸和机型判断了不同的机型，从此高枕无忧，知道19年的9月份。

## 页面问题

### frame
iphone11系列伴随着iOS13发布，问题在19年9月份集中爆发。
首先iPhoneX和iphoneXSM的判断是可以用到iphonePro的，因为他们的屏幕尺寸相同，布局不会出现问题。
但是我之前自作聪明的使用机型信息判断XR的方式放在iphone11上肯定不生效，因此iphone11出现的问题最多。

我一开始还是想通过机型判断，但是[维基](https://www.theiphonewiki.com/wiki/Models)上都找不到新的机型信息,而且适配的问题不能等太久，用户就是上帝。

于是我又将目标转向了屏幕本身，终于知道了比较简单能判断刘海屏的方法。

```
if (@available(iOS 11.0, *)) {
                        if (UIApplication.sharedApplication.keyWindow.safeAreaInsets.bottom > 0) {
                           //code
                        }
                    }
```

看，很简单吧，为什么用`bottom`而不用`top`,因为top返回的在其他机型上也是大于0的，如果我们用top就需要写一个20，但是我感觉判断的时候写一个魔法数有点怪异，所以还是决定用"bottom"大于0来判断了。

同样碰到一些底部和顶部的视图，我是用了这样

```
if (@available(iOS 11.0, *)) {
        h+=UIApplication.sharedApplication.keyWindow.safeAreaInsets.bottom;
    }
```
这样就能保证底部安全区域。

### 图片资源
对于图片资源如果要追求最好的效果，应该是让UI针对刘海屏出一些图，但是目前UI同事有优先级更高的事情，所以没办法，只能自己弄。

一般的图片问题，可以通过拉伸的方式解决，把图片弄到`xcassets`里，然后手动拉伸，美滋滋。

但是祖传代码就是会有这样一个问题，总会有一些看起来奇怪但是有合理的代码。

```
backgroudView.backgroundColor = [UIColor colorWithPatternImage:lastImage];
```

我有一个背景图，它的是图片平铺绘制的。按照逻辑我们是不是可以先拉伸再平铺如同对付一般图片一样。但是很抱歉，是不行的。

我这边的解决办法是使用`UIGraphics`重绘图片，而且不会根据机型判断，全部重绘。


### SearchBar不得不说的事情
我们总想app比别人好看，这是正常需求。

所以我们对SearchBar下手了，我们根据对`subViews`的各种判断，修改了左侧放大镜图标，修改了右侧的取消按钮。终于iOS13制裁了我们，判断逻辑失效了，层级变化很大，如果按照之前的方式。我的一个searchBar的样式修改，代码长度要达到250行，这到底是弄啥咧。这样如果一旦iOS对其进行了修改，我们又需要重新适配一遍。还有个好消息iOS13.1与iOS13的发布间隔不到一周，没人敢保证这块层级会不会再改。

于是有一个折中的方案，searchBar的背景我自己定义，但是左侧和右侧的按钮使用系统原生，效果还是可以的。

## 代码问题

由于iOS13相对于此前有很多修改，在一些使用keyPath获取系统未暴露的属性时会crash，目前发现的是`placeholder.textColor`。

其他还会有问题，目前还没有完全适配完，后续会继续补充


