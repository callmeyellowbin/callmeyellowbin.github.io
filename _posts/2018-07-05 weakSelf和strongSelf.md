---
layout:     post
title:      weakSelf和strongSelf
#subtitle:   学习对象初始化
date:      2018-07-05
author:     Hoben Wong
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - iOS
    - Objective-C
---
在很多项目中，都能看见一个`block`前后有`weakSelf`和`strongSelf`的身影，究竟为什么要这样写呢？参考以下博客，弄明白了：
1. [到底什么时候才需要在ObjC的Block中使用weakSelf/strongSelf](http://blog.lessfun.com/blog/2014/11/22/when-should-use-weakself-and-strongself-in-objc-block/)
2. [透彻理解block中weakSelf和strongSelf](https://www.jianshu.com/p/ae4f84e289b9)

####1.Block
OC里面的`Block`是个很实用的语法，如果和`GCD`一起使用，就可以很方便地实现并发、异步等任务。但是如果使用不当，就会产生**循环引用**的问题。原因：`Block` 持有 `self`，同时`self`也持有了`Block`。
####2.weakSelf
对此的解决方案是：在传入`Block`之前，将`self`转换成`weak automatic`的变量，这样`Block`就不会出现对`self`的强引用。如果在`Block`执行完成之前，`self`被释放的话，则`weakSelf`也会变成`nil`。
```
__weak __typeof__(self) weakSelf = self;
dispatch_async(queue, ^{
    [weakSelf doSomething];
});
```
这样，`Block`就不会对`self`持续引用，造成循环引用的bug了。
####3.strongSelf
但是，如果`weakSelf`要在`Block`持续使用的话，如果这时候`self`被突然释放了，`weakSelf`就会被释放，导致`weakSelf`的工作未完成，比如：
```
__weak __typeof__(self) weakSelf = self;
dispatch_async(queue, ^{
    //执行到这里不会被释放
    [weakSelf doSomething];

    //执行到这里有可能会被释放
    [weakSelf doOtherThing];
});
```
这时候`strongSelf`就要登场了，它可以确保在`Block`内，`strongSelf`不会被释放，同时可以保证不会造成循环引用的情况。
```
__weak __typeof__(self) weakSelf = self;
dispatch_async(queue, ^{
    __strong __typeof__(self) strongSelf = weakSelf;    

    //执行到这里不会被释放
    [strongSelf doSomething];
    [strongSelf doOtherThing];
});
```
使用`strongSelf`之后，指针连带关系`self`的引用计数还会增加。但是`strongSelf`是在`Block `里面，生命周期也只在当前`Block `的作用域。

所以，当这个`Block `结束，`strongSelf`随之也就被释放了。同时也不会影响`Block `外部的`self`的生命周期。
