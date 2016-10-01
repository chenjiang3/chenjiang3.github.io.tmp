---
layout: post
title: Swift闭包3-逃逸闭包
date: 2016-09-31 10:32:24.000000000 +09:00
---
### 引言
&emsp;&emsp;本篇将介绍swift中一种特殊的闭包-逃逸闭包（escape closure）。

### 逃逸闭包
&emsp;&emsp;当一个传入函数的闭包在函数执行结束之后才会被调用，这样的闭包就叫做逃逸闭包。如果一个函数的参数有一个逃逸闭包，可以在参数前加@escaping关键字来修饰。

&emsp;&emsp;一个闭包是逃逸必要的条件是这个闭包需要存储在函数外部。举个例子，很多异步操作的函数往往会传入一个complete handler作为异步操作完成后的回调。当这个异步函数开始执行的时候，会开启一个异步操作，然后这个函数就直接结束了，此时，传入的闭包还没有被执行，实际上这个回调需要在异步操作完成后才会被执行。这种情况下这个回调的闭包需要定义成逃逸闭包，因为它在函数调用结束之后才会被执行。比如下面的例子：

```
var completionHandlers: [() -> Void] = []
func someFunctionWithEscapingClosure(completionHandler: @escaping () -> Void) {
    completionHandlers.append(completionHandler)
}
```
someFunctionWithEscapingClosure以一个completionHandler作为参数，这个参数会被保存在函数外部的completionHandlers数组中，这时这个闭包是一个逃逸闭包，所以需要添加@escaping关键字去修饰，否则会有编译错误。

&emsp;&emsp;逃逸闭包如果需要使用对象的变量或常量的时候，必须显示指明self，如果是普通的闭包，可以直接使用对象的变量或常量。比如下面的例子：

```
var completionHandlers: [() -> Void] = []
func someFunctionWithEscapingClosure(completionHandler: @escaping () -> Void) {
    completionHandlers.append(completionHandler)
}

func someFunctionWithNonescapingClosure(closure: () -> Void) {
    closure()
}

class SomeClass {
    var x = 10
    func doSomething() {
        someFunctionWithEscapingClosure { self.x = 100 }
        someFunctionWithNonescapingClosure { x = 200 }
    }
}

let instance = SomeClass()
instance.doSomething()
print(instance.x)
// 输出 "200"

completionHandlers.first?()
print(instance.x)
// 输出 "100"
```
在这个例子中，第一个print输出200，因为当调用doSomethig的时候，someFunctionWithNonescapingClosure会直接调用闭包{x = 200},此时instance.x变成200，当completionHandlers.first?()之后，someFunctionWithEscapingClosure传入的闭包才会真正执行，此时instance.x变成100.可以看到，逃逸闭包必须显示指明self，而普通的闭包可以直接使用x。
