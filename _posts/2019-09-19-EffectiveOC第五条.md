---
layout:     post
title:      用枚举表示状态、选项、状态码
subtitle:   Effective OC 第五条
date:       2019-09-19
author:     二狗子
header-img: img/home-bg-o.jpg
catalog: true
tags:
    - Effective OC
---
# 用枚举表示状态、选项、状态码

### C++11 枚举标准

> OC是基于C语言的，所以OC也具有枚举类型。系统框架中频繁用到此类型，然而开发者容易忽视它。由于C++11标准扩充了枚举的特性，所以最新版系统框架是用了“强类型”枚举，OC也得益于C++11标准。

其实C++11是11年正式公布，距现在也很多年了，这一章书中大量提到了C++11，这里我稍微整理一下C++11中的枚举标准，来便于理解后续的OC枚举。

#### 强类型

C++11是用了“强类型”枚举，即`enum class`。可以理解为显式声明了枚举的类型。文中提到这样就可以“向前声明枚举变量”了，为什么？其实与分配空间有关，我们需要为声明的枚举变量分配足够的空间，在之前由于我们没有明确它的类型，所以我们不能提前知道它会占用多少空间，所以无法向前声明，现在我们需要显式声明枚举，所以也就提前知道了它的类型，所以可以向前声明了。

这里多提一句，在C++11之前，枚举值的作用域是全局的，也就是说不会被`{}`所限制。在C++11之后，作用域就限制在了`{}`内。

同样枚举编号可以自定义，后续每个枚举递增1。同样枚举也是有取值范围的，总结就是

> 枚举的上限是 大于最大枚举量的 最小的 2 的幂，减去 1；
> 枚举的下限有两种情况：一、枚举量的最小值不小于 0，则枚举下限取 0；二、枚举量的最小值小于 0，则枚举下限是小于最小枚举量的最大的 2 的幂，加上 1。

详细可以看[这里](https://www.runoob.com/w3cnote/cpp-enums-intro-and-strongly-typed.html)

### OC枚举

#### NS_ENUM

```
typedef NS_ENUM(NSUInteger, EOCConnectionState) {
    EOCConnectionStateDisconnected = 1,
    EOCConnectionStateConnecting,
    EOCConnectionStateConnected,
};
```

这里可以看到底层数据类型为NSUInteger，并且我们typedef重新定义了枚举类型，使代码更加简洁。
这种枚举经常与swtich搭配使用。

### NS_OPTIONS

```
typedef NS_OPTIONS(NSUInteger, EOCOptions) {
    EOCOptionsLeft      = 1 << 0, //0001
    EOCOptionsTop       = 1 << 1, //0010
    EOCOptionsRight     = 1 << 2, //0100
    EOCOptionsBottom    = 1 << 3, //1000
};
```
NS_OPTIONS是以移位的形式初始化的。因此经常使用“按位或”，在设置圆角的时候，我们会用到类似的枚举。

原文中的枚举书写形式已经被新的书写形式取代，所以我这边代码都是以新的形式书写，同时原文中提到了兼容问题，我感觉现在这个意义不大，所以就没有细读。
