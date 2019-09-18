#多用字面量语法少用与之等价的方法
###什么是字面量语法
```NSString *address = @"望京北路";```
文中提到了两个优点**缩减源代码长度**和**易读**
###缩短源代码长度
这个在文中仅提了一句，但是真的缩短了长度了吗?
这边我调用了这样两段代码

```
NSString *stringLiteral = @"我是字面量";
```
```
NSString *stringOther = [NSString stringWithFormat:@"我是与之等价"];
```
将文件转化为cpp文件后变成这样
![](https://tva1.sinaimg.cn/large/006y8mN6ly1g73rzvnxo8j30ki04imxz.jpg)
我们看到了第二种方法还需要调用stringwithFormat方法，确实是长了（其实我不知道我这整对不对。）
###易读性
#####数值型
仅举例

```
    NSNumber *intNumber = @1;
    
    NSNumber *floatNumber = @2.5f;

```
#####数组型
主要问题体现在创建和取用

* 创建

```NSArray *somePersons = @[@"小明",@"小红"];```

* 取用

```NSString *man = somePersons[0];```

* 问题

```NSArray *somePersons = @[@"小明",nil,@"小红"];```
如果使用字面量创建数组时，其中对象含有**nil**，那么会报错“数组中元素必须是个OC对象”。

```
id o1 = @"asdfasdf";
id o2 = nil;
id o3 = @"asdfsad1"; 
    
NSArray *arr1 = [NSArray arrayWithObjects:o1,o2,o3, nil];

NSArray *arr2 = @[o1, o2 ,o3];
```
如上，在创建数组时，我们故意将中间一个对象设置为nil，结果是可以编译成功，但是arr1中只包含o1，运行到arr2时会报错，原因是“将一个空对象加入数组中”。

结论是，使用字面量语法更安全，what？，其实想想也是，如果数组中加入了一个nil说明程序出了问题，还有比崩溃更能直观体现问题的方式吗？
#####字面量字典
```
NSDictionary *dic1 = [NSDictionary dictionaryWithObjectsAndKeys:@"小明",@"name",@5,@"age", nil];
    
NSDictionary *dic2 = @{@"name":@"小明",@"age":@5};
```
字面量字典的情况与数组类似，同样需要内里对象为OC对象，如果加一个nil进去，会及时报错。
#####可变数组和字典
主要体现在取数据上

```
NSString *arrString = arr1[0];
    
NSString *dicString = dic2[@"name"];
```

###缺陷
字面量语法所创建出来的对象必须属于Foundation框架才行。



