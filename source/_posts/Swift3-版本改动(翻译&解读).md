---
title: Swift3-版本改动(翻译&解读)
date: 2016-05-23 15:46:21
top: true
tags:
 - iOS
 - Swift
---
** 学习Swift2.2到3.0的改动,主要是对已经通过的提案进行解读.**
<!-- more -->
<The rest of contents | 余下全文>
## SE-0001:允许关键字作为参数名(Allow (most) keywords as argument labels)
<font size = '4', color = #FF8C00 >___Before___</font>

之前(swift2.1之前)我们在定义方法名的时候,如果遇到了关键字比如`in`,就需要加上单引号来避免混淆,因此有很多开发者选择了其他非关键字的单词作为参数名.
<font size = '4', color = #FF8C00 >___After___</font>

由于参数名在Swift方法中扮演者至关重要的作用,要做到简单易懂,现在我们在定义function时,可以使用类似`in`, `repeat`,和`defer`这种关键字:
``` swift
func touchesMatching(phase: NSTouchPhase, in view: NSView?) -> Set<NSTouch>
```

Swift3.0取消了___几乎所有___关键字做参数名的限制,除了`inout`,`let`,`var`,这三个如果必须要用到的话,还是需要像以前一样加单引号:
``` swift
func addParameter(name: String, `inout`: Bool)
```

## SE-0002:移除科里化(Removing currying func declaration syntax)  
<font size = '4', color = #FF8C00 >___Before___</font>

在之前的版本,我们可以这样写一个方法:
``` swift
func curried(x: Int)(y: String) -> Float {
    return Float(x) + Float(y)!
}
```

然后可以给它传递一个函数,使之变成一个可以接受一个参数的另外一个方法:
``` swift
let plusFive = curried(5)
//调用:
plusFive(7)     //结果12.0
```

<font size = '4', color = #FF8C00 >___After___</font>

在Swift3.0中,不再有科里化的专属表示方式,而是用一个返回值是另一个函数的方法表示:
``` swift
func curried(x: Int) -> (Int) -> Float {
    return { (y: Int) -> Float in
        return Float(x) + Float(y)
    }
}
```

值得注意的是:在`Swift3.0`中,我们需要强调的是函数式编程思想,即:___函数是一等公民___,换种说法就是函数可以被看作是一个值来传递,建议看看喵神翻译的`Functional Swift-函数式Swift`.

## SE-0003:从方法参数限制中去除var(Removing var from Function Parameters)
<font size = '4', color = #FF8C00 >___Before___</font>

我们在方法中使用传入的参数时,默认是不能修改的,除非在参数名前加入限定词`var`或`inout`:
``` swift
func foo(i: Int) {
  i += 1 // 错误
}

func foo(var i: Int) {
  i += 1 // OK
}

func foo(inout i: Int) {
  i += 1 // OK
}
```

这里需要科普一下,var和inout虽然都可以改变传入参数的值,它们的区别在于,var虽然可以修改参数,但是修改的相当于传入参数的一个copy,并不会影响传入的参数本身(这里可以理解为浅复制),inout则可以修改,可以理解为传入的是一个地址:
``` swift
func doSomethingWithVar(var i: Int) {
  i = 2 // 不会影响传入参数的原值,但是可以在方法生命周期内任意修改
}

func doSomethingWithInout(inout i: Int) {
  i = 2 // 会直接影响传入参数的原值
}

var x = 1
print(x) // 1

doSomethingWithVar(x)
print(x) // 1

doSomethingWithInout(&x)
print(x) // 2
```

<font size = '4', color = #FF8C00 >___After___</font>

3.0中移除了var这个限制词,因为根本没有必要这样做,如果需要改变一个值,完全可以创建一个变量等于传入的值,然后便可以对其进行任意的修改:
``` swift
func foo(i: Int) {
  var i = i
}
```

## SE-0004:移除++和--运算符(Remove the ++ and -- operators)
<font size = '4', color = #FF8C00 >___Before___</font>

之前的用法:

``` swift
let a = ++x
let b = x++
let c = --x
let d = x--
```

然而,这些操作符的结果值总是被人混淆忽略,他们有着自己的优势和劣势:

<font size = '3'>___优势___</font>

+ 简洁--他们要比`x += 1`或者`x.advance()`这种函数更加简洁,尤其是在迭代的时候.当需要有返回值时,`+=`并不能胜任,除非返回的是`void`.
+ 接地气--这种运算符和C语言及其扩展语言(C++,OC,C#,JAVA,JS等)都能很好的衔接,从其他语言转Swift的更容易接受.

<font size = '3'>___劣势___</font>

+ 难于理解--这种运算符增加了学习Swift的负担,尤其是在Swift是你接触的第一种编程语言的时候.
+ 并没有比`+=`简洁多少.
+ Swift拥有足够多的像`for-in`,`enumerate`,`map`的函数,对于++这种需求并没有那么大,没有他们一样可以流畅的编写代码.
+ Bulabulabulabula....

<font size = '4', color = #FF8C00 >___After___</font>

总之就是不再用了,但是为了兼容C++等语言,还是保留了编译能力.


## SE-0005:将OC的接口更好的转换成Swift接口(Better Translation of Objective-C APIs Into Swift)
这部分主要介绍了工程师们设计API的出发点,如果想要了解深入,可以看看WWDC16的视频,或者访问[Swift API Design Guidelines](https://swift.org/documentation/api-design-guidelines/),不过个人不建议看,没什么鸟玩意,只是介绍了一下设计API的思路.

建议SE-0005,SE-0006,SE-0023一起看,看proposal原文档.

<font size = '4', color = #FF8C00 >___Before & After___</font>

这一部分还是要日常开发中慢慢积累,举个例子:
``` swift
// 修改之前
let content = listItemView.text.stringByTrimmingCharactersInSet(
   NSCharacterSet.whitespaceAndNewlineCharacterSet())
// 修改之后
let content = listItemView.text.trimming(.whitespaceAndNewlines)
```

再比如:
``` swift
+[NSNumber numberWithBool:];
//改为:
NSNumber(bool: true)
```

还有,拿UIBezierPath举例子:
``` swift
class UIBezierPath : NSObject, NSCopying, NSCoding {
  convenience init(ovalInRect: CGRect)
  func moveToPoint(_: CGPoint)
  func addLineToPoint(_: CGPoint)
  func addCurveToPoint(_: CGPoint, controlPoint1: CGPoint, controlPoint2: CGPoint)
  func addQuadCurveToPoint(_: CGPoint, controlPoint: CGPoint)
  func appendPath(_: UIBezierPath)
  func bezierPathByReversingPath() -> UIBezierPath
  func applyTransform(_: CGAffineTransform)
  var empty: Bool { get }
  func containsPoint(_: CGPoint) -> Bool
  func fillWithBlendMode(_: CGBlendMode, alpha: CGFloat)
  func strokeWithBlendMode(_: CGBlendMode, alpha: CGFloat)
  func copyWithZone(_: NSZone) -> AnyObject
  func encodeWithCoder(_: NSCoder)
}

改为:

class UIBezierPath : NSObject, NSCopying, NSCoding {
  convenience init(ovalIn rect: CGRect)
  func move(to point: CGPoint)
  func addLine(to point: CGPoint)
  func addCurve(to endPoint: CGPoint, controlPoint1 controlPoint1: CGPoint, controlPoint2 controlPoint2: CGPoint)
  func addQuadCurve(to endPoint: CGPoint, controlPoint controlPoint: CGPoint)
  func append(_ bezierPath: UIBezierPath)
  func reversing() -> UIBezierPath
  func apply(_ transform: CGAffineTransform)
  var isEmpty: Bool { get }
  func contains(_ point: CGPoint) -> Bool
  func fill(_ blendMode: CGBlendMode, alpha alpha: CGFloat)
  func stroke(_ blendMode: CGBlendMode, alpha alpha: CGFloat)
  func copy(with zone: NSZone = nil) -> AnyObject
  func encode(with aCoder: NSCoder)
}
```

简单地说,修改的主要思路就是,方法名里涉及到参数的基本上都转换为参数的外部参数,如果通过参数名完全可以知道这个函数是干什么的,那么方法名将会省略相关文字比如:
``` swift
func appendString(_: NSString)
//改为:
func append(_: NSString)
```

不再赘述.

## SE-0006:将API设计思想运用到标准库(Apply API Guidelines to the Standard Library)
这部分和SE-0005合并一起,内容很多,总结起来就是命名更加突出面向协议了,例如`enum MyType: ErrorType`现在要写成`enum MyType: ErrorProtocol`.这种类似的Type结尾的协议名,都显式的命名为-Protocol了.详见[命名修改](https://github.com/apple/swift-evolution/blob/master/proposals/0006-apply-api-guidelines-to-the-standard-library.md).

<font size = '4', color = #FF8C00 >___Before & After___</font>

简单写几个:
``` swift
sort() => sorted(), sortInPlace() => sort().

reverse() => reversed().

enumerate() => enumerated().

SequenceType.minElement() => .min(), .maxElement() => .max().
```

## SE-0007:移除C语言格式的for循环(Remove C-style for-loops with conditions and incrementers)
C语言格式的for循环在Swift中并没有很多的使用,并且并不符合Swift的风格.
与之相比,for-in这种更具有Swift风格的已经可以替代C格式的语言,删除C格式的for循环将简化Swift语言,并且常见的++,--这种已经从Swift中被淘汰了.

<font size = '4', color = #FF8C00 >___Before___</font>

举个例子
``` swift
for var i = 0 ; i < 10 ; i++ {
    print(i)
}
```

以及
``` swift
var array = [10,20,30,40,50]
for(var i=0 ; i < array.count ;i++){
    println("array[i] \(array[i])")
}
```

<font size = '4', color = #FF8C00 >___After___</font>

``` swift
for i in 0..<len {
  dst[i] = src[i]
}
```

简单不废话.

## SE-0008:添加一个lazy flatMap为可选序列(Add a Lazy flatMap for Sequences of Optionals)
<font size = '4', color = #FF8C00 >___Before___</font>

Swift标准库中有两个版本的flatMap,其中一种是对普通序列的展开:
``` swift
[1, 2, 3]
  .flatMap { n in n..<5 }
// [1, 2, 3, 4, 2, 3, 4, 3, 4]
```

另一种是对可选序列的展开:
``` swift
(1...10)
  .flatMap { n in n % 2 == 0 ? n/2 : nil }
// [1, 2, 3, 4, 5]
```

然而,只有第一种情况存在着lazy实现:
``` swift
[1, 2, 3]
  .lazy
  .flatMap { n in n..<5 }
// LazyCollection<FlattenBidirectionalCollection<LazyMapCollection<Array<Int>, Range<Int>>>>

(1...10)
  .lazy
  .flatMap { n in n % 2 == 0 ? n/2 : nil }
// [1, 2, 3, 4, 5]
```

<font size = '4', color = #FF8C00 >___After___</font>

用标准库中已经存在的类型来描述,flatMap的功能可以用,map-filter-map链来实现:
``` swift
extension LazySequenceType {

  @warn_unused_result
  public func flatMap<T>(transform: Elements.Generator.Element -> T?)
    -> LazyMapSequence<LazyFilterSequence<LazyMapSequence<Elements, T?>>, T> {
      return self
        .map(transform)
        .filter { opt in opt != nil }
        .map { notNil in notNil! }
  }
}
```
