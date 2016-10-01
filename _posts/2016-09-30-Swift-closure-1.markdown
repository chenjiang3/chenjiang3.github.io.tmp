---
layout: post
title: Swift闭包1-基本概念
date: 2016-09-30 9:32:24.000000000 +09:00
---
### 引言
&emsp;&emsp;这篇文章开始主要讲解Swift中闭包（Closures）的一些知识点。本文是这个系列的第一篇。

### 闭包的基本概念
&emsp;&emsp;闭包的就是匿名函数，别的语言也有类似的概念，在Objective-c中就是block，在c++中就是lambdas表达式。闭包可以获取闭包所在的上下文的变量和常量，并在闭包内部使用。在Swift中，全局函数和嵌套函数是特殊的闭包，有3种类型。

1. 全局闭包：定义在全局，不捕获外部变量
2. 局部闭包：定义在函数内部，会捕获函数里面的变量
3. 匿名的闭包表达式：可以捕获上下文的变量

### 闭包的语法
&emsp;&emsp;Swift中闭包的定义方式如下。

```
{ (parameters) -> return type in
    statements
}
```
&emsp;&emsp;in关键字前面的是闭包的申明部分，类似函数的申明，包括参数的申明和返回值的申明；in后面的部分是闭包的实现。比如swift自带的array的sort函数，调用sort的时候需要传一个compare的闭包进去，代码如下：

```
reversedNames = names.sorted(by: { (s1: String, s2: String) -> Bool in
    return s1 > s2
})
```

### 根据上下文推测类型
&emsp;&emsp;上面的sort方法传入的是一个很标准的闭包表达式，在swift中，闭包表达式可以更简单，swift可以根据上下文推断出闭包表达式中参数的类型，以及返回值的类型。比如上面的sort函数，传入的闭包肯定是比较array里面的元素，所以闭包参数的类型肯定和array里面的元素类型一样（在这里是String类型），同样的，作为比较函数，返回值肯定是Bool类型。所以，参数类型和返回值类型都可以根据上下文推断出来，那么在闭包里面就可以直接省略。更简介的代码如下：

```
reversedNames = names.sorted(by: { s1, s2 in return s1 > s2 } )
```
&emsp;&emsp;上面的闭包只需要知道s1和s2就行了，类型可以推断出来。同理，返回值也可以直接省略。

### 省略return关键字
&emsp;&emsp;当闭包的实现只有一行代码的时候，可以省略return关键字。比如在上面的代码中sorted(by:) 函数的定义明确表示需要闭包返回一个Bool类型的值，同时闭包的实现只有一行代码，显然可以判断，这一行代码肯定返回一个Bool类型的值，所以return关键字可以直接省略。

### 简写参数名
&emsp;&emsp;swift可以采用更简洁的方式来定义参数名，例如采用$0,$1,$2等等来引用闭包的参数，如果采用这种参数，in关键字也可以省略掉，最后闭包表达式可以非常简单，如下：

```
reversedNames = names.sorted(by: { $0 > $1 } )
```
$0表示第一个参数，$1表示第二个参数。

### 运算符函数
&emsp;&emsp;还有一种更简单的表达方式，因为String定义了大于（>）运算符.

```reversedNames = names.sorted(by: >)```
