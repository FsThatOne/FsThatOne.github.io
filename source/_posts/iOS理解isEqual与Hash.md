---
title: iOS理解isEqual与Hash
date: 2018-03-13 14:53:40
top: false
tags:
 - iOS
 - OC
---
** 理解iOS中isEqual与Hash的作用与自定义方式**
<!-- more -->
<The rest of contents | 余下全文>

# `isEqual:`方法
### isEqual的作用:
```
判断两个对象是否相等
```
### 和"=="的区别?为什么而存在:
```
对于基本数据类型(值类型等),"=="运算符比较的是值是否相同;而对于对象类型,则是比较两者的内存地址是否一致(即是否是同一对象).
```
`PS: 上述==运算符的说明只适用于OC,JAVA这种不支持运算符重载的语言(支持运算符重载的语言有C++等).`
### 所以要理清==和isEqual分别比较的是什么?
他们一个比较的是指针,一个比较的是对象本身,下面用一个例子来说明:
```ObjectiveC
UIColor *color1 = [UIColor colorWithRed:0.5 green:0.5 blue:0.5 alpha:1.0];
UIColor *color2 = [UIColor colorWithRed:0.5 green:0.5 blue:0.5 alpha:1.0];
NSLog(@"color1 == color2 => %@", color1 == color2 ? @"YES" : @"NO");
NSLog(@"[color1 isEqual:color2] => %@", [color1 isEqual:color2] ? @"YES" : @"NO");
```

打印结果如下:

```
color1 == color2 => NO
[color1 isEqual:color2] => YES
```

从上面的例子可以看出, ==运算符只是简单地判断是否是同一个对象,即指针是否指向同一地址, 而isEqual方法可以判断对象是否相同, 例如UIColor对象表示的color是否相同.

### 如何重写自己的isEqual方法?
向上面例子,UIColord的equal方法已经被s实现了,同样在Cocoa Framework中被实现的还有:
```objectivec
NSString isEqualToString;
NSDate isEqualToDate;
NSArray isEqualToArray;
NSDictionary isEqualToDictionary;
NSSet isEqualToSet;
......
```
PS: 更多请参考 [Equality](https://nshipster.com/equality/)
但对于自定义的类型来说,我们想要实现isEqual,就需要自己重写isEqual方法,给出如何判断两个类是否相等的方法,下面给出正确的姿势:
1. 首先,我们定义一个自己的类`Person`
```Objectivec
@interface Person : NSObject

@property (nonatomic, copy) NSString *name;
@property (nonatomic) NSDate *birthday;

@end
```
2. `Person`类实现的`isEqual:`方法如下:
```Objectivec
- (BOOL)isEqual:(id)object {
    if (self == object) {
        return YES;
    }
    if (![object isKindOfClass:[Person class]]) {
        return NO;
    }
    return [self isEqualToPerson:(Person *)object];
}

- (BOOL)isEqualToPerson:(Person *)person {
    if (!person) {
        return NO;
    }
    BOOL haveEqualNames = (!self.name && !person.name) || [self.name isEqualToString:person.name];
    BOOL haveEqualBirthdays = (!self.birthday && !person.birthday) || [self.birthday isEqualToDate:person.birthday];
    return haveEqualNames && haveEqualBirthdays;
}
```
3. 上面的代码很好理解,两个person名字和生日一样,我们才认为两个person是equal的.
这时你可能会想,光一个`isEqual:`方法就可以解决两个类型是否相等的判断了,那`Hash`有什么用呢?下面我们来看看`Hash`的作用.


# Hash方法
这个要从HashTable(哈希表)这种数据结构说起.
1. 首先,假设我们要从一个数组中找一个特定的元素,最糟糕的情况是我们需要遍历这个数组才能找到对应元素.在数组未排序的情况下,查找一个元素的**时间复杂度**是`O(n)`,n为数组的长度.
2. 为提高查找速度,一种新的存储结构**哈希表**出现了,当新的元素进入表中时,会分配一个`hash`值,以标记它在集合中的位置,通过这个`hash`值,就能够在`O(1)`的时间复杂度内找到该元素,但如果多个元素拥有同一个哈希值,那就不能保证`O(1)`的时间复杂度了.
```
分配的这个hash值(即用于查找集合中成员的位置标识), 就是通过hash方法计算得来的, 且hash方法返回的hash值最好唯一
```
3. 和数组相比,**哈希表**的查找顺序是先通过Hash值找到元素的存储位置,如果元素和哈希值一一对应,则直接取元素,否则则用遍历的方式找到目标元素.那么问题来了,我们重写的`Hash`方法,什么时候被调用呢?

### Hash方法什么时候被调用?
举个栗子~~
```objectivec
Person *person1 = [Person personWithName:kName1 birthday:self.date1];
Person *person2 = [Person personWithName:kName2 birthday:self.date2];

NSMutableArray *array1 = [NSMutableArray array];
[array1 addObject:person1];
NSMutableArray *array2 = [NSMutableArray array];
[array2 addObject:person2];
NSLog(@"array end -------------------------------");

NSMutableSet *set1 = [NSMutableSet set];
[set1 addObject:person1];
NSMutableSet *set2 = [NSMutableSet set];
[set2 addObject:person2];
NSLog(@"set end -------------------------------");

NSMutableDictionary *dictionaryValue1 = [NSMutableDictionary dictionary];
[dictionaryValue1 setObject:person1 forKey:kKey1];
NSMutableDictionary *dictionaryValue2 = [NSMutableDictionary dictionary];
[dictionaryValue2 setObject:person2 forKey:kKey2];
NSLog(@"dictionary value end -------------------------------");

NSMutableDictionary *dictionaryKey1 = [NSMutableDictionary dictionary];
[dictionaryKey1 setObject:kValue1 forKey:person1];
NSMutableDictionary *dictionaryKey2 = [NSMutableDictionary dictionary];
[dictionaryKey2 setObject:kValue2 forKey:person2];
NSLog(@"dictionary key end -------------------------------");
```
输出结果如下:
```objectivec
array end -------------------------------
hash = 7809196951631946839
hash = 7809196951631946839
hash = 7809191961023760480
hash = 7809191961023760480
set end -------------------------------
dictionary value end -------------------------------
hash = 7809196951631946839
hash = 7809196951631946839
hash = 7809191961023760480
hash = 7809191961023760480
dictionary key end -------------------------------
```
从结果可以看出:
```
hash方法只有对象被添加在Set,或者充当Dictionary的Key时才会调用Hash方法,因为这两种场景下不会有重复元素/Key,需要做去重处理,那么就需要给出判断依据,Hash方法就是这个依据的规则.
```

### Hash方法和判断相等的关系?
hash方法主要是用于在Hash Table查询成员用的, 那么和我们要讨论的isEqual()有什么关系呢?

为了优化判等的效率, 基于hash的NSSet和NSDictionary在判断成员是否相等时, 会这样做:
1. 集合成员的hash值是否和目标hash值相等, 如果相同进入Step 2, 如果不等, 直接判断不相等.
2. hash值相同(即Step 1)的情况下, 再进行对象判等, 作为判等的结果.
简单说就是:
```
hash值相等是判断对象相同的必要不充分条件.
```

### 如何重写?
`NSObject`已经实现了类的`Hash`方法,得到的结果是这个对象的内存地址.当我们将两个"Equal"的对象放入一个`Set`时,会发现,虽然这两个对象通过我们重写的`isEqual:`方法判断相等,但在Set中还是以不同的元素存在:
```objectivec
Person *person1 = [Person personWithName:kName1 birthday:self.date1];
Person *person2 = [Person personWithName:kName1 birthday:self.date1];
NSLog(@"[person1 isEqual:person2] = %@", [person1 isEqual:person2] ? @"YES" : @"NO");

NSMutableSet *set = [NSMutableSet set];
[set addObject:person1];
[set addObject:person2];
NSLog(@"set count = %ld", set.count);
```
输出:
```
[person1 isEqual:person2] = YES
set count = 2
```
isEqual相等的两个对象都加入到了NSSet中(set count = 2), 证明两者的Hash值不同,于是我们需要最佳实践:

`大神Mattt Thompson在Equality中给出的结论就是:`
```
In reality, a simple XOR over the hash values of critical properties is sufficient 99% of the time(对关键属性的hash值进行位或运算作为hash值)
```
于是,我们对重写上述的Person类的`Hash`方法进行重写:
```objectivec
- (NSUInteger)hash {
    return [self.name hash] ^ [self.birthday hash];
}
```
更多关于位运算的内容,可参考[Implementing Equality and Hashing](https://www.mikeash.com/pyblog/friday-qa-2010-06-18-implementing-equality-and-hashing.html).

