---
layout: post
title: Swift闭包2-尾闭包
date: 2016-09-30 10:32:24.000000000 +09:00
---
### 引言
&emsp;&emsp;上一片文章讲解了swift中闭包的基本概念，以及基本的语法，这篇文章主要讲解一下swift中尾闭包。

### 什么是尾闭包
&emsp;&emsp;假设有这样一个函数：这个函数的最后一个参数是一个闭包，并且，这个闭包表达式很长。如果按照正常的方式来调用这个函数的话，需要在参数里指明这个函数的参数名，然后后面跟着一个闭包表达式。比如下面这种函数：

```
func someFunctionThatTakesAClosure(closure: () -> Void) {
    // function body goes here
}
```
正常的使用方式如下

```
// 正常的调用方式:

someFunctionThatTakesAClosure(closure: {
    // closure's body goes here
})
```
可以发现someFunctionThatTakesAClosure(closure:)最一个参数是一个闭包，那么我们可以用另外一种方式去调用这个函数。

```
// 用尾闭包的方式调用:

someFunctionThatTakesAClosure() {
    // trailing closure's body goes here
}

```
Swift的Array的sort函数也可以采用尾闭包的方式来调用，比如[上一篇](https://chenjiang3.github.io/2016/09/Swift-closure-1/)文章对字符串数组排序的方法。

```
reversedNames = names.sorted() { $0 > $1 }
```
&emsp;&emsp;如果一个函数只有一个参数，并且这个参数是一个闭包，那么在使用这个函数的时候可以省略括号，比如上面的sorted函数可以写成这样：

```
reversedNames = names.sorted { $0 > $1 }
```
