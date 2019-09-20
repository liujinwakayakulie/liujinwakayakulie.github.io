---
layout:     post
title:      理解“对象等同性”这一概念
subtitle:   Effective OC 第八条
date:       2019-09-20
author:     二狗子
header-img: img/home-bg-o.jpg
catalog: true
tags:
    - Effective OC
---
# 理解“对象等同性”这一概念

### OC中的等同性

提到“等同性”很自然的就会想到`==`，但是在OC中几乎不会这么使用，OC中使用的基本都是`isEqualXXX:`。因为OC对象基本都是指针类型，所以不能使用`==`比较，实际上需要比较的是指针指向的值，以OC为例，我们会这样比较
```[arrString isEqualToString:dicString]```

返回值是一个BOOL类型。文中以NSObject为例子介绍了判断等同性的的两个方法

* `- (BOOL)isEqual:(id)object;`
* `@property (readonly) NSUInteger hash;`

文中在描述hash方法是，是一个对象方法，但是观察NSObject.h文件，已经没有这个方法了。先说下文中提到的方法实现思路：当且仅当“指针值”完全相等时，两个对象才想等。如果自定义的对象中需要覆写着两个方法，需要理解这两个方法的结果关系。如果`isEqual`返回`YES`，`hash`一定相等。如果`hash`相等，`isEqual`不一定返回`YES`。

### isEqual

```
- (BOOL)isEqual:(id)object {
    if (self == object) {
        return YES;
    }

    if ([self class] != [object class]) {
        return NO;
    }
    EOCPerson *otherPerson = (EOCPerson *)object;
    if (![_name isEqualToString:otherPerson.name]) {
        return NO;
    }
    //依次比较属性
    return YES;
}
```
比较流程大概是指针->类->每个属性。

其中如果指针相同直接返回相同。

### hash

#### 第一种方法

```
- (NSUInteger)hash {
    return 1137;
}
```
这个为什么会影响性能，我打算再后续针对hash单独开一张，有一篇[文章](https://www.jianshu.com/p/bbeec2a570aa)在这里mark一下。

#### 第二种方法
```
- (NSUInteger)hash {
    NSString *stringToHash = [NSString stringWithFormat:@"%@:%@",_name,_address];
    return [stringToHash hash];
}
```
这个需要创建字符串，同样存在性能问题。

#### 第三种方法

```
- (NSUInteger)hash {
    NSUInteger nameHash = [_name hash];
    NSUInteger addressHash = [_address hash];
    return nameHash ^ addressHash;
}
```
这个方法有两个作用，效率高，同时将哈希码位数限定在一定范围内，但是依然会碰撞。

### 特定类的等同性判定方法

这里提到的特定类是集合类，他们的判定方法是以`isEqualToClassName:`的形式。

如果自己编写判定方法，需要覆写`isEqual`方法。规则：如果受测的参数与接受该消息的对象都属于同一个类，那么调用自己编写的判定方法，否则交给超类来判断。这里的超类就是父类。具体的逻辑如下，自己的判定方法参考上文。

```
- (BOOL)isEqual:(id)object {
    if ([self class] == [object class]) {
        return  [self isEqualToPerson:(EOCPerson *)object];
    } else {
        return [super isEqual:object];
    }
}
```

### 等同性判定的执行深度

这里主要是两个方面

* 根据整个对象判断等同性
* 根据几个字段判断（如ID）

### 容器中可变类的等同性

这里主要讨论了可变部分对等同性判断的影响。

如果我们将可变对象存入容器类中，由于对象可变，因此hash码是有可能改变的，这就会出现错误。因此应该避免哈希码是根据可变部分生成的。

会有这样一个场景，NSSet本身是会去重的，我们如果存入两个指向不同字符串的NSString对象进去，然后将这两个字符串修改成指向同一个字符串，这样一个NSSet中就存在了两个一样的字符串。然后我们将这个NSSet copy,copy出来的NSSet对象只包含一个字符串。

具体的代码稍后补充 Mark住



