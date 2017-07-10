---
layout: post
title: CATransaction
date: 2016-10-15 10:32:24.000000000 +09:00
---

&emsp;&emsp;事务（CATransaction）实际上是Core Animation中用来包含一系列属性动画集合的机制，任何用指定事务去改变可以做动画的图层属性都不会立刻发生变化，而是当事务提交的时候才开始用一个动画过度到新值。

&emsp;&emsp;事务通过CATransaction类来管理，这个类没有属性和实例方法，并且也不能用+alloc和+init方法创建它，而是用+begin和+commit分别来入栈和出栈。

&emsp;&emsp;任何可以做动画的图层属性都会添加到当前栈顶的事务，可以通过+setAnimationDuration：方法来设置当前事务的动画时间，并且可以通过+animationDuration：方法来获取事务的动画时间，默认是0.25秒。Core Animation会在每个run loop周期中自动开始一次新的事务，如果你没有指定新的事务，在这个run loop中的任何属性变化都会搜集起来加到这个默认的事务当中，然后做一次0.25秒的动画。

&emsp;&emsp;那CATransaction到底如何使用呢，请看下面的例子。

```objc
- (void)onChangeColor:(id)sender {
    [CATransaction begin];
    [CATransaction setAnimationDuration:3];

    CGFloat red = arc4random() / (CGFloat)INT_MAX;
    CGFloat green = arc4random() / (CGFloat)INT_MAX;
    CGFloat blue = arc4random() / (CGFloat)INT_MAX;
    self.colorLayer.backgroundColor = [UIColor colorWithRed:red green:green blue:blue alpha:1.0].CGColor;

    [CATransaction commit];
}
```

当调用-onChangeColor：的时候，随机生成一个颜色，并修改colorLayer的背景颜色，可以看到他的颜色变化是渐变的。所以，使用CATransaction很简单，在+begin和+commit之间加入需要动画的代码，并且可以设置动画时间（通过+setAnimationDuration：）。

&emsp;&emsp;可以看到，CATransaction的+begin:和+commit:和UIView的+beginAnimations:context:和+commitAnimations非常类似。实际上UIView的+beginAnimations:context:和+commitAnimations内部就是设置了CATransaction。UIView还有另外一个方法+animateWithDuration:animations:，一样的，在这个方法的内部也是设置了CATransaction，这样才能起到动画的效果。

&emsp;&emsp; UIView的动画方法允许你在动画结束的时候提供一个完成的block，同样的，CATransaction也有类似的接口，+setCompletionBlock:就可以提供一个动画结束的回调，我们来调整下上面的例子，让colorLayer的颜色修改完成之后旋转一个角度，代码如下：

```
- (void)onChangeColor:(id)sender {
    [CATransaction begin];
    [CATransaction setAnimationDuration:3];

    [CATransaction setCompletionBlock:^{
        CGAffineTransform t = self.colorLayer.affineTransform;
        t = CGAffineTransformRotate(t, M_PI_4);
        self.colorLayer.affineTransform = t;
    }];

    CGFloat red = arc4random() / (CGFloat)INT_MAX;
    CGFloat green = arc4random() / (CGFloat)INT_MAX;
    CGFloat blue = arc4random() / (CGFloat)INT_MAX;
    self.colorLayer.backgroundColor = [UIColor colorWithRed:red green:green blue:blue alpha:1.0].CGColor;

    [CATransaction commit];
}
```

当调用-onChangeColor：的时候，colorLayer的颜色会从起始值变化到新的值，并且是渐变的，当颜色变化结束后，又会旋转一个角度。

&emsp;&emsp;注意到一个现象，这里颜色变化的时间明显要比旋转的时间慢。这是因为颜色变化是在当前的事务中进行的，在这里这个事务的动画时间是3秒，而且旋转是在当前事务出栈之后才被执行的，也就是在系统默认的事务中执行的，默认的事务执行时间是0.25秒，所以旋转动画的时间要快。
