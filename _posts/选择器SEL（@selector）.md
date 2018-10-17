---
layout:     post
title:      选择器SEL（@selector）
#subtitle:   学习对象初始化
date:      2018-07-16
author:     Hoben Wong
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - iOS
    - Objective-C
    - 底层知识
---
先来介绍一下下面几个相互依赖的概念：
- 方法：由程序代码构成，是对象的实现的一部分，相对于操作本身重点强调执行操作的方法对于面向对象程序设计很重要，因为不同的对象可能具有执行相同操作的不同方法。

- 消息：让对象执行操作的请求，对象确定将使用哪个方法来执行操作。可以把相同的消息发送给不同的对象，以及产生不同的结果。

- 选择器：确定要发送给对象的消息，并且接受消息的对象将会使用选择器来选择调用哪个方法。

OC提供了`SEL`数据类型，用来声明存储选择器的变量
```
SEL aSelector = @selector(update)

id result1 = [obj update];
id result2 = [obj performSelector: @selector(update)];
id result3 = [obj performSelector: aSelector];
```
`@selector`会先寻找父类的方法，再寻找子类的方法，找到之后继续调用，没有找到之后会报地址寻找出错。

而在发送消息之前验证一个对象响应了选择器，则调用以下方法
```
- (BOOL) respondsToSelector: (SEL) selector
```
这个方法很常见，因为我们在委托的时候需要用到，当委托对象需要调用方法的时候，则需要先验证这个方法是否存在于受委托对象里面，再进行调用。
```
//列数
- (NSUInteger) columnCount
{
    if ([self.delegate respondsToSelector: @selector(columnCountInWaterFallLayout:)]) {
        return [self.delegate columnCountInWaterFallLayout: self];
    }
    else
        return HobenDefaultColumnCount;
}
```
####工作原理
参考[iOS中的SEl和IMP到底是什么](https://www.jianshu.com/p/4a09d5ebdc2c)

先介绍两个定义：

- SEL：类成员方法的指针，但不同于C语言中的函数指针，函数指针直接保存了方法的地址，但SEL只是方法编号。

- IMP：一个函数指针,保存了方法的地址。

IMP和SEL有什么关系呢？

每一个继承于`NSObject`的类都能自动获得`runtime`的支持。在这样一个类中，有一个`isa`指针，指向该类定义的数据结构体，这个结构体是由编译器编译时为类（继承于`NSObject`）创建的。在这个结构体中有包括了：指向其父类定义的指针以及`Dispatch Table`。`Dispatch Table`是一张SEL和IMP的对应表。

![](https://upload-images.jianshu.io/upload_images/8407639-5c647b37f852879e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

也就是说方法编号SEL最后还是要通过`Dispatch table表`寻找到对应的IMP，IMP就是一个函数指针，然后执行这个方法。

在编译的时候, 只要有方法的调用, 编译器都会通过 selector 来查找,所以 (假设 addObject 的 selector 为 12)
```
 [myObject addObject:yourObject]; 
```
将会编译变成
```
objc_msgSend(myObject, 12, yourObject);
```
这里，`objec_msgSend()`函数将会使用` myObject` 的` isa 指针`来找到` myObject `的类空间结构并在类空间结构中查找 `selector 12` 所对应的方法。

如果没有找到，那么将使用指向父类的指针找到父类空间结构进行 `selector 12` 的查找。

如果仍然没有找到，就继续往父类的父类一直找，直到找到为止， 如果到了根类 NSObject 中仍然找不到，将会抛出异常。

我们可以看到, 这是一个很动态的查找过程。类的结构可以在运行的时候改变,这样可以很容易来进行功能扩展，Objective-C 语言是动态语言, 支持动态绑定。

#####1.怎么获得方法编号？
@selector()就是取类方法的编号
```
SEL methodId=@selector(func1);
```
#####2.编号获取之后怎么执行对应的方法？
```
[self performSelector: methodId withObject: nil];
```
#####3.怎么通过编号获取方法？
```
NSString *methodName = NSStringFromSelector(methodId);
```
#####4.IMP怎么获得和使用？
```
IMP methodPoint = [self methodForSelector: methodId];
methodPoint();
```
#####5.SEL的存在意义是什么？
有了SEL这个中间过程之后，我们可以对一个编号和什么方法映射做些操作，也就是说我们可以用一个SEL指向不同的函数指针，这样就可以完成一个方法名在不同时候执行不同的函数体。

另外，可以将SEL作为参数传递给不同的类执行，也就是说我们某些业务中，只知道方法名，但需要根据不同的情况让不同的类执行的时候，SEL可以帮助到我们。
