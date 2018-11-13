---
title: React Native 学习历程 - 小技巧
date: 2017-04-10 9:46:21
top: false
tags:
- React Native
- iOS
---
** 这个系列记录了我正式开始学习React Native的历程与心得,以及一些踩过的坑,本篇将讲述学习React Native中遇到的一些小技巧,以及ES6的一些巧妙的语法 **
<!-- more -->
<The rest of contents | 余下全文>

# 延展操作符(...)

这个`...`操作符（也被叫做延展操作符 － spread operator）已经被ES6数组 支持。它允许传递数组或者类数组直接做为函数的参数而不用通过apply。

```js
var people=['Wayou','John','Sherlock'];
//sayHello函数本来接收三个单独的参数人妖，人二和人三
function sayHello(people1,people2,people3){
  console.log(`Hello ${people1},${people2},${people3}`);
}
//但是我们将一个数组以拓展参数的形式传递，它能很好地映射到每个单独的参数
sayHello(...people);//输出：Hello Wayou,John,Sherlock 
//而在以前，如果需要传递数组当参数，我们需要使用函数的apply方法
sayHello.apply(null,people);//输出：Hello Wayou,John,Sherlock
```
在React Native中:

```js
render() {
    var params = {name: 'Lily', age: 18, sex: 'man'};
    return (
        <MyView {...params}/>
    )
}
```

当然,它也可以和普通的XML属性混合使用，需要同名属性，后者将覆盖前者：

```js
var props = { foo: 'default' };
var component = <Component {...props} foo={'override'} />;
console.log(component.props.foo); // 'override'
```

# export和export default
当我们写完一个组件之后,为达到复用的目的,一般都会把它导出以提供给其他页面使用。
导出的关键字是:

```js
export default
```
没有加`default`时比如:

```js
export class Template{}
```
至于两者的区别,稍后再说,我们可以在单个js文件中恒明多个组件或者函数,比如:

```js
export class Template{}
export class AnotherTemplate{}
export function sum(a,b) {}
```
这样在其他文件引用时，需要使用{}符号且组件名称必修和class名称一样，像这样子：

```js
import {Template,AnotherTemplate,sum} from './components/Templates';
```
而加default时，例如：

```js
export default class Template{}
```
其他文件引用时,可以不用加大括号:

```js
import Template from './components/Templates';
```
当然,我们也可以给他起个别名,以方便使用:

```js
import ThatOne from './components/Templates';
```
PS:<font size = '4' color = '#464686'> ___每个文件中只能有一个default导出,可以有多个非默认导出___ </font>
导出多个组件或者方法时:

```js
import Template,{AnotherTemplate, sum} from './components/Templates';
```
当然还有一种写法:

```js
import * as exportObject from './components/Templates';
```
这种写法是把Templates中的所有(无论是否default)export的属性、组件、方法组成一个对象,这个对象拥有所有导出的属性、方法、组件,使用时直接`exportObject.xxx`就好了.


