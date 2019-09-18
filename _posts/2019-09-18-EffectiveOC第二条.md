---
layout:     post
title:      在类的头文件中尽量少引入其他头文件
subtitle:   Effective OC 第二条
date:       2019-09-18
author:     二狗子
header-img: img/home-bg-o.jpg
catalog: true
tags:
    - Effective OC
---
# 在类的头文件中尽量少引入其他头文件
### 类
在Xcode中，类基本上都是这样创建的
![](https://tva1.sinaimg.cn/large/006y8mN6ly1g73qdjm3lmj30kc0ehgo5.jpg)

这里创建了一个名字为EOCPerson的类，他是NSObject的子类，使用OC语言，创建之后我们会得到**EOCPerson.h**和**EOCPerson.m**两个文件。

![EOCPerson.h](https://tva1.sinaimg.cn/large/006y8mN6ly1g73qippjjnj30he06c74v.jpg)
![EOCPerson.m](https://tva1.sinaimg.cn/large/006y8mN6ly1g73qiuwzs8j30cg02w0sr.jpg)

我们看到由于我们创建的是一个NSObject的子类，所以自动引入了Foundation框架。如果创建的是与view相关的子类，可能需要引入UIKit框架。
### 头文件
上文中提到了h和m文件，那么h文件就是头文件，这个EOCPerson的头文件已经默认引入了Foundation。EOCPerson有两个属性name和address，都是NSSring类型，如果我们想自定义一个类型作为EOCPerson的属性，应该怎么做呢。如果我们直接加上这行代码
```@property (nonatomic, strong) EOCEmployer *employer;```

其实不会报错找不到EOCEmployer类，而是会报错strong必须修饰一个object type。
我们可以在Foundation的引入下面加上这样一行代码
```import "EOCEmployer.h"```
如果直接加上文中提到这样不够优雅，因为在变异EOCPerson类的文件时，不需要知道EOCEmployer的全部内容，只需要知道有一个类叫EOCEmployer。因此我们将上面一行代码改成
```@class EOCEmployer;```
但是我们肯定会在m文件中使用到EOCEmployer，因此为了配合我们需要在m文件的头部引入
```import "EOCEmployer.h"```
这样做的目的是为了将引入头文件的时机延后，因为在编译头文件的时候我们不需要知道其他类的具体内容，这样就减少了编译时间。
### 向前声明解决的问题

* 减少编译时间
* 防止“循环引用”

关于“循环引用”，如果EOCEmploer的头文件以import引入EOCPerson类，那么在编译的时候会报错，EOCPerson中会报错EOCEmployer类型问题。
### 现实情况
但是所有的情况都是头文件**@class**，实现文件**import**。实际上不是的，如果头文件中必须要使用属性、实例变量、协议，那就需要在头文件中**import**了，遇到这种情况怎么继续保持优雅呢，这里挖个坑，在第27条会填上。
