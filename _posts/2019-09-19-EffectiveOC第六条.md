---
layout:     post
title:      理解“属性”这一概念
subtitle:   Effective OC 第六条
date:       2019-09-19
author:     二狗子
header-img: img/home-bg-o.jpg
catalog: true
tags:
    - Effective OC
---

# 理解“属性”这一概念

### 实例变量
直接请出今天的代码

```

@interface EOCPerson : NSObject
{
    @public
        NSString *_Name;    //所有引入可以访问
    @protected
        NSNumber *_Age;     //当前类和子类可以访问
    @private
        NSString *_Sex;     //仅当前类访问
}

@end
```

这个在19年看来可能是一个不怎么常见的写法了，但是有的时候理解这个对于属性还是很有必要的。实例变量其实是有四个级别的，但是这三个最常用就列在这里了，注释中已经表明了他们的作用域（嗯，作用域这个词很装逼）。

#### 实例变量内存访问

基本来说就是会将访问`_Name`变量的代码转换成偏移量，这个偏移量是硬编码，表示该变量距离存放对象内存区域的其实地址有多远。

但是这样就会有一个问题，如果这个偏移量在编译期就计算好了，那么我下次在最前面添加实例变量`_address`，如果不重新编译，那么就会访问错误的实例变量。

但是OC中显然没有遇到这个问题，为什么，因为偏移量的计算是在运行时计算的。我们还可以在运行时添加实例变量，这叫做`ABI`。

防止这个问题还有一个办法，我们不直接访问实例变量，而是通过存取方法。那么问题又来了，我们平时怎么定义存取方法的，又怎么使用存取方法的呢。

### property

```@property (nonatomic, copy) NSString *name;```

这个是我们平时最常写的属性声明，注意这里是属性。但是即便如此，没我们依然没有像这样写代码。

```
    [person setName:@"小明"];
    NSString *personName = [person name];
```

很显然，第一行是设置，第二行是读取。一般我们会直接使用`.name`。
第一行代码对我们来说是做了三件事情，都是在编译期完成。

* 生成实例变量`_name`
* 生成setter方法
* 生成getter方法

默认生成实例变量的规则是“`_`+属性名”。如果想要自己定义实例变量的名字可以使用`@synthesize`来自定义。

同样如果不想让编译期自动生成存取方法可以使用`@dynamic`阻止其自动生成。

以上我们解释了为什么一开始生成实例变量的时候都加了`_`以及`@ property `做了什么事情。接下来我们来解释`(nonatomic, copy)`。

### attribute

#### 原子性

默认情况下，编译器所合成的方法通过锁定机制保证其原子性。

此处锁的留坑

* nonatomic 消耗资源小，不安全，setter不加锁
* atomic 消耗资源大，安全，setter加锁

`注`：atomic也只是相对安全，比如线程A向地址中写入数据，线程B等到线程A写完后修改了其中的数据，线程A再次访问时，获取的就不是线程A之前写入的数据了。

#### 读写权限

* readwrite 读写
* readOnly 只读，一般用在对外暴露只读属性，实现文件读写属性

#### 内存管理语义

这个地方留个坑，这段可以在的读书笔记中单独开两章

* assign 用来形容纯量类型
* strong 拥有关系
* weak 非拥有关系
* unsafe_unretained 非拥有关系
* copy 拥有关系，常用在不可变类型

#### 方法名

如果不想使用系统生成的方法，我们可以自定义方法名。

```
@property (nonatomic, copy, getter=isMan, setter=sexChange:) NSString *sex;
```
getter方法自定义是比较常用的，setter方法自定义不太常用。

上述的含义解释之后，我们什么情况下会用到呢。文中提到了如果自己实现存取方法需要实现此前声明的attribute，还有一种情况是在初始化的是否，比如下面：

```
- (id) initWithName:(NSString *)name {
    if (self = [super init]) {
        _name = [name copy];
    }
    return self; 
}
```

我们本可以使用直接赋值和调用点语法来实现，但是我们使用了实例变量和此前声明的copy方法。因为在初始化方法是，要遵循属性定义中的attribute。但是调用点语法不就是直接使用set方法，也可以实现copy方法啊，为什么不用呢，因为在init方法中不使用set方法会在后边写。



