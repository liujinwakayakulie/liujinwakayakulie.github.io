# 在对象内部尽量直接访问实例变量

### 前言
对象对外的时候，我们肯定需要使用属性来访问，但是对象内部肯定需要对属性进行操作，我们应该通过什么方式呢，书中建议尽量使用实例变量，同意、同意、同意。

### 正文
```
//对外暴露两个对象方法
- (NSString *)nowSex;
- (void)changeSex:(NSString *)sex;
```

```
//点语法版本
- (NSString *)nowSex {
    return self.sex;
}

- (void) changeSex:(NSString *)sex {
    self.sex = sex;
}
```
```
//实例变量版本
- (NSString *)nowSex {
    return _sex;
}

- (void) changeSex:(NSString *)sex {
    _sex = sex;
}
```

从上面看起来就书写来说其实差不多是不是，但是因为点语法的特性，所以还是连带出了一些不同。

* 点语法使用时经过了OC的“方法派发”（11条填坑），直接访问实例变量速度比较快。
* 使用set方法势必要受attribute的影响，这些是多余的开销
* 如果直接访问实例变量，不会触发KVO，如果有这方面的需求就无法满足
* 由于set和get方法中可以添加断点，因此点语法方便调试

针对以上的问题，文中提出了一个折中方案，写入实例变量时，使用set方法来做，访问实例变量时，使用get方法来做。优点就是提高读取速度，并且在写入时可以确保实现attribute。但是折中方案依然有需要注意的地方对不对。

####折中方案注意点一

尽量不要在初始化方法中调用set方法。

情景再现

```
EOCPerson
- (instancetype)init
{
    if (self = [super init]) {
        self.name = @"";
    }
    return self;
}
```

```
EOCPerson子类重写setName方法
- (void)setName:(NSString *)name {
    if (![name isEqualToString:@"小红"]) {
        [NSException raise:NSInvalidArgumentException format:@"红是我爱的人"];
    }
    self.name =  name;
}
```
```
报错
Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '红是我爱的人'
```

我们在父类中初始化方法中使用点语法把属性name初始化为空，这里调用了setName。在子类中我们重写了setName，当我们new一个新的子类对象时，就会报错，因此初始化时需要使用实例变量

#### 折中方案注意点二

惰性初始化

需要惰性初始化的时候我们需要通过get方法访问属性。惰性初始化的意思就是在对象初始化的时候我们不初始化对应属性，只有在需要访问的时候才会在get方法中进行初始化操作，一般用于属性初始化过于复杂的情况。因为如果此时直接访问实例变量，是空的。

