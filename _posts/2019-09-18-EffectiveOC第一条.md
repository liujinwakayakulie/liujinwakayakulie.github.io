---
layout:     post
title:      了解Objective-C语言的起源
subtitle:   Effective OC 第一条
date:       2019-09-18
author:     二狗子
header-img: img/home-bg-o.jpg
catalog: true
tags:
    - Effective OC
---
##了解Objective-C语言的起源
###起源
OC是由[Smalltalk](https://zh.wikipedia.org/wiki/Smalltalk)演化来的，与C++和Java的“函数调用”不同，OC使用“消息结构”。
###何为消息结构
首先是以“函数调用”和“消息结构”的对比为引子来开始写的，他们的关键区别在于执行代码是**运行环境**决定还是**编译器**决定。二者又有什么区别呢，简单来说**编译器**是将代码转化成设备能理解的机器码，这个时候代码还是“死的”。运行时就是已经将代码装载到了内存中，代码“活了”。详细可以看[这里](https://blog.csdn.net/weiwenhp/article/details/8107203)。那么是不是C++的代码只能会在编译器决定呢，不是的，这又引出了**多态**。关于多态以及虚方法表，可以看[这里](https://blog.csdn.net/dan15188387481/article/details/49667389)。
由上面就可以看出两种方式的不同，”消息结构“总是在运行时查找方法，而”函数调用“则会根据具体情况来定。
###OC中的堆栈
>    NSString *address = @"望京北路";
> 
>    NSString *anotherAddress = address;

以上面两行代码举例子。堆栈主要在两方面表现不同**管理方式**和**分配方式**。
#####分配方式
首先address和anotherAddress都是`NSString *`指针，他们被分配在**栈**上，被分配了两块内存。他们指向同一个NSString实例`@“望京北路”`,这个实例被分配在**堆**上的。
#####管理方式
对于栈，会自动清理。对于堆，需要程序员手动分配和释放内存。OC是C的超集，但是在写OC的时候也没有malloc和free，因为这里已经被抽象成了“引用计数”。
#####结语
堆栈的概念不会只是这么简单，详细的可以看[这里](https://www.jianshu.com/p/c8e1d91dda99)
