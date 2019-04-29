### 一. 绘图简介

- Core Graphics是UIKit下的主要绘图系统，在绘制自定义视图的需求中经常会被使用，他的数据结构和函数通过前缀CG来识别。

- 视图可以通过子视图、图层或实现`drawRect:`方法来表现内容，但最好不要混用，只使用其中一种方法。

- 由于像素依赖于目标，所以2D绘图不能操作单独的像素，需要从上下文（Context）读取。

### 二. 绘图基础

#### 1. 基本概念
- context：上下文，iOS绘图需要传一个上下文，这个Context在重写UIView的`drawRect:`方法里调用`UIGraphicsGetCurrentContext()`获取。

- path：路径，在iOS绘图中，可以想象为拿着一支笔去画图，画几条线或几个点从而形成一个路基纪念馆，然后可以利用路径去填色、描边。

- stroke、fill：描边和填充，每个路径都需要填充或者描边之后才能在视图看见，可以设置他们的颜色、粗细、渐变、连接样式等等。

- 画图可以使用默认路径画，也可以单独创建path画图，对应画图的api不完全相同，是两组名称相似的api，重用方法如下：

```
CGContextMoveToPoint    // 设置起点
CGContextClosePath      // 连接起点和当前点
CGPathCreateMutable     // 类似于CGContextBeginPath，创建新路径
CGPathMoveToPoint       // 类似于CGContextMoveToPoint，画笔移动，声明开始画线
CGPathAddLineToPoint    // 类似于CGContextAddLineToPoint，设置终点
CGPathAddCurveToPoint   // 类似于CGContextAddCurveToPoint，画曲线
CGPathAddEllipseInRect  // 类似于CGContextAddEllipseInRect，画圆
```

#### 2. 画图步骤

画图的步骤分为四个步骤：获取Context→设置线条属性→设置路径→填充或描边路径，在继承UIView的对象view的`drawRect:`方法实现。
```
- (void)drawRect:(CGRect)rect {
    [super drawRect:rect];
    
    // 获取Context
    CGContextRef ctx = UIGraphicsGetCurrentContext();
    
    // 设置线条属性
    CGContextSetStrokeColorWithColor(ctx, [UIColor blackColor].CGColor);//设置线的颜色
    CGContextSetLineWidth(ctx, 2);    // 设置线的宽度
    CGContextSetLineCap(ctx, kCGLineCapRound);    // 设置线的起始端的样式
    CGContextSetLineJoin(ctx, kCGLineJoinRound);    // 设置线的连接样式
    
    // 设置路径
    CGContextMoveToPoint(ctx, 50, 50);    // 设置起点坐标
    CGContextAddLineToPoint(ctx, 200, 100);    // 设置终点坐标
    
    // 填充或描边路径
    CGContextStrokePath(ctx);
}
```
所得的效果如下：
![](https://upload-images.jianshu.io/upload_images/8407639-5c103c62aa2126a7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 3. 更多类型的图

##### 1) 折线
折线即在普通画线方法的基础上多加几个点，也可调用`GContextAddLines(CGContextRef c,
​    CGPoint *points, size_t count)`方法：

```
- (void)drawLines:(CGContextRef)ctx {
    CGPoint lines[] = {
        CGPointMake(10.f, 90.f),
        CGPointMake(70.f, 60.f),
        CGPointMake(130.f, 90.f),
        CGPointMake(190.f, 60.f),
        CGPointMake(250.f, 90.f),
        CGPointMake(310.f, 60.f)
    };
    CGContextAddLines(ctx, lines, sizeof(lines) / sizeof(lines[0]));

    // 填充或描边路径
    CGContextStrokePath(ctx);
}
```

![](https://upload-images.jianshu.io/upload_images/8407639-61e0ce7577037cfd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 2) 虚线
虚线绘制的原理即在绘画直线的时候通过控制交替绘制，即什么时候绘画，什么时候跳过绘画来实现，需要调用`CGContextSetLineDash(CGContextRef c, CGFloat phase,
​    CGFloat * __nullable lengths, size_t count)`，参数意义如下：

- context: 上下文

- phase: 初次绘制的时候跳过几个点

- lengths: 数组，描述如何交替绘制

- count: 数组长度

------

- length数组举例：

假如length为{10, 10}，代表先绘制10个点，再跳过10个点，如此反复：

![](https://upload-images.jianshu.io/upload_images/8407639-fe718512818f48f4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

假如length为{10, 20, 10}，代表先绘制10个点，再跳过20个点，再绘制10个点，跳过10个点，绘制20个点，如此反复：

![](https://upload-images.jianshu.io/upload_images/8407639-b63ab90abdaf70e2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

------

- phase数组举例：

以length为{10, 10}为例，phase代表着初次绘制时跳过几个点，phase为5时如下：

![](https://upload-images.jianshu.io/upload_images/8407639-8607023c68f72024.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

------

虚线具体代码如下：

```
- (void)drawLines:(CGContextRef)ctx {
    CGPoint lines[] = {
        CGPointMake(10.f, 90.f),
        CGPointMake(70.f, 60.f),
        CGPointMake(130.f, 90.f),
        CGPointMake(190.f, 60.f),
        CGPointMake(250.f, 90.f),
        CGPointMake(310.f, 60.f)
    };
    CGContextAddLines(ctx, lines, sizeof(lines) / sizeof(lines[0]));
    
    //设置虚线
    CGFloat length[] = {10, 10};
    CGContextSetLineDash(ctx, 5, length, sizeof(length) / sizeof(length[0]));

    // 填充或描边路径
    CGContextStrokePath(ctx);
}
```

##### 3) 圆/多边形

```
- (void)drawSharp:(CGContextRef)ctx {
    CGContextSetFillColorWithColor(ctx, [UIColor blackColor].CGColor);
    
    // 画椭圆，若长宽相等就是圆
    CGContextAddEllipseInRect(ctx, CGRectMake(80, 250, 50, 50));
    
    // 画矩形，长宽相等就是正方形
    CGContextAddRect(ctx, CGRectMake(170, 250, 50, 50));
    
    /*** 以下方法二选一 ***/
    
    // 描边
    CGContextStrokePath(ctx);
    
    // 填充
    CGContextFillPath(ctx);
}
```

![描边](https://upload-images.jianshu.io/upload_images/8407639-0711f5b64b05a091.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![填充](https://upload-images.jianshu.io/upload_images/8407639-f3222034795668ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 4) 文字

```
- (void)drawText:(CGContextRef)ctx {
    // 文字样式
    UIFont *font = [UIFont systemFontOfSize:18.f];
    NSDictionary *dict = @{
                           NSFontAttributeName : font,
                           NSForegroundColorAttributeName : [UIColor blackColor]
                           };
    
    [@"Hoben" drawInRect:CGRectMake(120.f, 350.f, 500.f, 50.f) withAttributes:dict];
}

```

##### 5) 图片

```
- (void)drawImage:(CGContextRef)ctx {
    UIImage *image = [UIImage imageNamed:@"image_example"];
    
    [image drawInRect:CGRectMake(0, 100.f, image.size.width, image.size.height)];
}
```

##### 6) 圆弧

画弧主要用以下方法：`CGContextAddArc`, `CGContextAddArcToPoint`, `CGContextAddCurveToPoint`（贝塞尔三次曲线）, `CGContextAddQuadCurveToPoint`（贝塞尔二次曲线）

---

方法一：CGContextAddArc

参数如下：

- CGContextRef c

- CGFloat x：圆心的x坐标

- CGFloat y：圆心的y坐标

- CGFloat radius：圆的半径

- CGFloat startAngle：开始弧度

- CGFloat endAngle：结束弧度

- int clockwise：0表示顺时针，1表示逆时针

画笔起始坐标为(x + radius, y)，开始弧度为0即从起始坐标开始画。

示例：

```
- (void)drawArc:(CGContextRef)ctx {
    // 顺时针半圆
    CGContextAddArc(ctx, 150.f, 100.f, 50.f, 0, M_PI, 0);
    
    CGContextStrokePath(ctx);
    
    // 圆
    CGContextAddArc(ctx, 150.f, 200.f, 50.f, 0, M_PI * 2, 0);
    
    CGContextStrokePath(ctx);
    
    // 逆时针半圆
    CGContextAddArc(ctx, 150.f, 300.f, 50.f, 0, M_PI, 1);
    
    CGContextStrokePath(ctx);
}
```

![](https://upload-images.jianshu.io/upload_images/8407639-7c837508db1ab67f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

-----

方法二：CGContextAddArcToPoint

这个方法一般用于绘制弧度，即通过起始点、途径的端点和半径控制这条圆弧的弧度，可以了解一下贝塞尔曲线的用法，参数如下：

- CGContextRef c

- CGFloat x1：端点1的x坐标

- CGFloat y1：端点1的y坐标

- CGFloat x2：端点2的x坐标

- CGFloat y2：端点2的y坐标

- CGFloat radius：半径

为了更好地理解该函数，该函数加入了辅助线：

```
- (void)drawArcToPoint:(CGContextRef)ctx {
    CGContextSetStrokeColorWithColor(ctx, [UIColor redColor].CGColor);
    CGContextSetLineWidth(ctx, 3);
    
    CGContextMoveToPoint(ctx, 60.f, 100.f);
    
    // 辅助线，可以注释
    CGContextAddLineToPoint(ctx, 160.f, 100.f);
    CGContextAddLineToPoint(ctx, 160.f, 180.f);
    CGContextMoveToPoint(ctx, 60.f, 100.f);
    
    CGContextAddArcToPoint(ctx, 160.f, 100.f, 160.f, 180.f, 50.f);
    CGContextDrawPath(ctx, kCGPathFillStroke);
}
```

所得的图像如下图所示，A(60,100), B(160,100), C(160, 180)，直线AB,BC为辅助线，得到的弧线即与直线AB,BC均相切的半径为50的弧。

![](https://upload-images.jianshu.io/upload_images/8407639-ac328007a5ea94e3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

去掉辅助线的曲线如下：

![](https://upload-images.jianshu.io/upload_images/8407639-e9943f70779528d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当|AB| = |BC| = 半径的时候，所形成的弧即为圆弧，由此，我们可以通过这个方法来画个圆：

```
- (void)drawArcToPoint:(CGContextRef)ctx {
    CGContextSetStrokeColorWithColor(ctx, [UIColor redColor].CGColor);
    CGContextSetLineWidth(ctx, 3);
    
    CGContextMoveToPoint(ctx, 80.f, 100.f);
    
    // 辅助线，可以注释
//    CGContextAddLineToPoint(ctx, 160.f, 100.f);
//    CGContextAddLineToPoint(ctx, 160.f, 180.f);
//    CGContextMoveToPoint(ctx, 80.f, 100.f);
    
    CGContextAddArcToPoint(ctx, 160.f, 100.f, 160.f, 180.f, 80.f);
    CGContextAddArcToPoint(ctx, 160.f, 260.f, 80.f, 260.f, 80.f);
    CGContextAddArcToPoint(ctx, 0.f, 260.f, 0.f, 180.f, 80.f);
    CGContextAddArcToPoint(ctx, 0.f, 100.f, 80.f, 100.f, 80.f);
//    CGContextDrawPath(ctx, kCGPathFillStroke);
    CGContextStrokePath(ctx);
}
```

![所得的圆恰好与手机屏幕相切](https://upload-images.jianshu.io/upload_images/8407639-201ca5daf0ee206f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

-----
方法三：CGContextAddCurveToPoint

这个函数是画三次贝塞尔曲线的，通过控制起点、途经点坐标和终点坐标来绘制弧线。关于贝塞尔曲线是怎么得来的，可以参考一下[这篇文章](http://www.html-js.com/article/1628)。参数如下：

- CGContextRef c

- CGFloat x1：控制点1的x坐标

- CGFloat y1：控制点1的y坐标

- CGFloat x2：控制点2的x坐标

- CGFloat y2：控制点2的y坐标

- CGFloat x：直线的终点的x坐标

- CGFloat y：直线的终点的y坐标

```
- (void)drawCurveToPoint:(CGContextRef)ctx {
    CGContextSetStrokeColorWithColor(ctx, [UIColor redColor].CGColor);
    CGContextSetLineWidth(ctx, 3);
    
    CGContextMoveToPoint(ctx, 80.f, 150.f);
    
    CGContextAddCurveToPoint(ctx, 160.f, 100.f, 210.f, 180.f, 260.f, 100.f);
    
    CGContextDrawPath(ctx, kCGPathFillStroke);
    CGContextStrokePath(ctx);
}
```

可以看到效果如下：

![](https://upload-images.jianshu.io/upload_images/8407639-1bc6372974328f3d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

------
方法四：CGContextAddQuadCurveToPoint，贝塞尔二次曲线

参数如下：

- CGContextRef c

- CGFloat cpx：控制点的x坐标

- CGFloat cpy：控制点的y坐标

- CGFloat x：直线的终点的x坐标

- CGFloat y：直线的终点的y坐标

```
- (void)drawQuadCurveToPoint:(CGContextRef)ctx {
    CGContextSetStrokeColorWithColor(ctx, [UIColor redColor].CGColor);
    CGContextSetLineWidth(ctx, 3);
    
    CGContextMoveToPoint(ctx, 80.f, 150.f);
    
    CGContextAddQuadCurveToPoint(ctx, 160.f, 100.f, 260.f, 100.f);
    
    CGContextDrawPath(ctx, kCGPathFillStroke);
    CGContextStrokePath(ctx);
}
```

这次我们减少了其中一个控制点，可以看到效果如下：

![](https://upload-images.jianshu.io/upload_images/8407639-b9cb62a9e96a72a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 五. Demo

我的Demo[在这里](https://github.com/callmeyellowbin/HobenCGImageDemo)，欢迎star~

#### 六. 参考文献

 [iOS开发——Core Graphics绘图](https://www.jianshu.com/p/fcf350eae255)

[iOS绘图—— UIBezierPath 和 Core Graphics](https://www.jianshu.com/p/8e6e960eea7d)

[iOS绘图系列二：画直线](https://blog.csdn.net/lcl130/article/details/41691907)

[贝塞尔曲线扫盲](http://www.html-js.com/article/1628)