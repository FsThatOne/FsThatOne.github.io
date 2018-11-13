---
title: iOS理解isEqual与Hash
date: 2018-03-13 14:53:40
top: false
tags:
 - iOS
 - OC
---
** iOS10通知的新API,和两个通知的扩展:NSNotification Service/Content Extension新特性的使用.**
<!-- more -->
<The rest of contents | 余下全文>

### 为什么会有isEqual方法
+ isEqual的作用:
```
判断两个对象是否相等
```
+ 和"=="的区别?为什么而存在:
```
对于基本数据类型(值类型等),"=="运算符比较的是值是否相同;而对于对象类型,则是比较两者的内存地址是否一致(即是否是同一对象).
```
`PS: 上述==运算符的说明只适用于OC,JAVA这种不支持运算符重载的语言(支持运算符重载的语言有C++等).`
+ 所以要理清==和isEqual分别比较的是什么?
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

### 如何重写自己的isEqual方法