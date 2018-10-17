---
layout:     post
title:      iOS中用动画处理解决跳闪情况
#subtitle:   学习对象初始化
date:      2018-08-13
author:     Hoben Wong
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - iOS
    - Objective-C
---
不知道大家在访问一些APP的图片时候有没有发现一些跳闪的情况：点击了一张图片之后，它会先显示缩略图，加载完图片之后再显示大图，由于缩略图和大图的大小差距过大，导致加载完成后，突然看到大图的情况，这种情况称为“跳闪”。
![跳闪](https://upload-images.jianshu.io/upload_images/8407639-63c331514a2b5f6c.gif?imageMogr2/auto-orient/strip)

###一.跳闪的引起
“跳闪”并不是一种bug，而是一种引起不好的用户体验的现象，由以下原因引起（需要同时满足以下条件）：

1.  缩略图和原图的尺寸差距非常悬殊
2.  点击缩略图的时候才开始加载原图
3.  缩略图变成原图的一瞬间缺乏必要的过渡动画

有很多APP为了避免跳闪情况的出现，在加载图片的过程中并没有选择显示动画，而是显示了`加载中`的GIF，这是一种解决跳闪的好方法，我称它为鸵鸟算法（手动滑稽）

而经过观察，微信票圈较少出现跳闪现象，这是由以下原因避免的：

1. 微信采用了`边滑动边加载`的策略，也就是说，有可能在你点击之前，这张图的原图已经加载完毕了，从而使得点击缩略图的时候，直接用原图的动画显示出来
2. 即便没有加载完毕，微信的缩略图和原图的尺寸也会`保持一致`，在加载过程中，会先显示缩略图（即经过压缩的原图），加载完毕后直接显示原图。

即便如此，在微信中也会有一种情况会出现跳闪：
- 竖型超长图，未加载完毕就点开查看，这时候，会先显示缩略图（一张高度经过极大压缩的图片），加载完成后则会直接显示超长图的顶部。

今天我们就来处理一下这种跳闪的现象：
![跳闪处理效果](https://upload-images.jianshu.io/upload_images/8407639-1f23989b4c4e687e.gif?imageMogr2/auto-orient/strip)
###二.设计思路
####1.竖型长图的查看
在上面的GIF中可以看到，竖型长图的过渡非常自然，这是由于缩略图和点击后映入眼帘的视图保持了一致，即两个必要条件：
1. 缩略图即原图截取的上半部分（无压缩）
2. 原图显示填充满整个屏幕

第一步是经过服务器实现的，不细说了，第二步则是通过设置`MWPhotoBrowser`的一个属性实现的：
```
photoBrowser.zoomPhotosToFill = YES;
```
这个属性能够确保查看原图的时候能够填充满整个屏幕。

####2.横型长图的查看
横向长图是比较容易出现跳闪的情况的，这是因为缩略图和显示的图片显示差距过大，直接跳转则显得极度不自然。
![缩略图](https://upload-images.jianshu.io/upload_images/8407639-ca27e602215ad8f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![原图](https://upload-images.jianshu.io/upload_images/8407639-8e2c385dcf14101f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这时候，则需要用到动画效果来修补，思路如下：
1. 原图加载完毕
2. 将缩略图压缩至原图的高度
3. 缩略图透明度渐变（由亮变暗）
4. 缩略图消失
5. 原图出现
6. 原图透明度渐变（由暗变亮）

首先怎么知道原图什么时候加载完成？`MWPhoto`可没有提供这个接口，那么我们就需要阅读一下源码，它到底什么时候加载图片了：
```
- (void)handleMWPhotoLoadingDidEndNotification:(NSNotification *)notification {
    id <MWPhoto> photo = [notification object];
    MWZoomingScrollView *page = [self pageDisplayingPhoto:photo];
    if (page) {
        if ([photo underlyingImage]) {
            // Successful load
            [page displayImage];
            [self loadAdjacentPhotosIfNecessary:photo];
            if ([self.delegate respondsToSelector:@selector(photoBrowserDidLoadSucceed:zoomScale:)]) {
                [self.delegate photoBrowserDidLoadSucceed:self zoomScale:page.zoomScale];
            }
        }
        else {
            // Failed to load
            [page displayImageFailure];
            if ([self.delegate respondsToSelector:@selector(photoBrowserDidLoadFailed:)]) {
                [self.delegate photoBrowserDidLoadFailed:self];
            }
        }
        // Update nav
        [self updateNavigation];
    }
}
```
在上面的代码可以看到，在`handleMWPhotoLoadingDidEndNotification`不仅可以加载图片，还可以顺便获得缩放的大小呢，好的，委托回调知道怎么用了，接下来就看看动画怎么实现了。

参考过很多关于动画的文章，尝试过很多种方案，最终我还是选择使用了继承`CABasicAnimation`的`CAAnimationGroup`。参考于[这篇文章](https://www.jianshu.com/p/02c341c748f9)，属性如下：
![](https://upload-images.jianshu.io/upload_images/8407639-d5547852434ee4f3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

好的，可以看到我们要用到的值有：
- transform.scale.y（压缩高度）
- opacity（透明度渐变）

那么，用动画将他们组合起来就可以了：
```
//添加动画
CAAnimationGroup *animations = [CAAnimationGroup animation];

CABasicAnimation *animation1 = [CABasicAnimation animationWithKeyPath: @"opacity"];
animation1.fromValue = [NSNumber numberWithFloat: 1.f];
animation1.toValue = [NSNumber numberWithFloat: .5f];

CABasicAnimation *animation2 = [CABasicAnimation animationWithKeyPath: @"transform.scale.y"];
animation2.fromValue = [NSNumber numberWithFloat: 1.f];
animation2.toValue = [NSNumber numberWithFloat: zoomScale];

animations.animations = @[animation1, animation2];
animations.duration = .2f;
animations.delegate = self;
animations.removedOnCompletion = NO;
animations.autoreverses = NO;
animations.fillMode = kCAFillModeBoth;
[self.helpImageView.layer addAnimation: animations forKey:@"AnimationKey"];
```
注意一下，以下这四句话都有自己的意义，具体可以看注释
```
//结束动画委托
animations.delegate = self;
//需要将其removedOnCompletion设置为NO,要不然fillMode不起作用
animations.removedOnCompletion = NO;
//不逆向
animations.autoreverses = NO;
//动画加入后开始之前,layer便处于动画初始状态,动画结束后layer保持动画最后的状态.
animations.fillMode = kCAFillModeBoth;
```
在缩略图动画完成之后（先压缩，后透明），令缩略图消失，并展现原图动画。需要用到动画结束之后的响应函数：
```
#pragma mark - CAAnimationDelegate

- (void)animationDidStop:(CAAnimation *)anim finished:(BOOL)flag {
    if (flag) {
        if ([self.helpImageView.layer animationForKey: @"AnimationKey"] == anim) {
            //恢复至原来的样子
            CAAnimationGroup *animations = [CAAnimationGroup animation];
            CABasicAnimation *animation1 = [CABasicAnimation animationWithKeyPath: @"opacity"];
            animation1.toValue = [NSNumber numberWithFloat: 1.f];
            CABasicAnimation *animation2 = [CABasicAnimation animationWithKeyPath: @"transform.scale.y"];
            animation2.toValue = [NSNumber numberWithFloat: 1.f];
            animations.animations = @[animation1, animation2];
            animations.fillMode = kCAFillModeBoth;
            [self.helpImageView.layer addAnimation: animations forKey: @"AnimationKey"];
            [self.helpImageView removeFromSuperview];
            [self.helpImageView.layer removeAllAnimations];
            self.photoBrowser.view.hidden = NO;
            CABasicAnimation *animation4 = [CABasicAnimation animationWithKeyPath: @"opacity"];
            animation4.duration = .2f;
            animation4.fromValue = [NSNumber numberWithFloat: .5f];
            animation4.toValue = [NSNumber numberWithFloat: 1.f];
            [self.photoBrowser.view.layer addAnimation: animation4 forKey:nil];
        }
    }
}
```
这样就可以按照之前提到的五个点来一步步实现了~
