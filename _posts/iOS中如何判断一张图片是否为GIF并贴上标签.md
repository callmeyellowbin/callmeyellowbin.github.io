在项目中，遇到一个问题，为GIF图片右下角加上一个动图的图标，于是百度了一下，发现普遍都是这样搞的：
```
NSData *data = [NSData dataWithContentsOfURL: [NSURL URLWithString: imageUrl]];
[data getBytes:&c length:1];
if (c == 0x47) {
    //如果是GIF，即便是长GIF也会显示动图
    self.longImageLabelText.text = @"动图";
}
```
跑在机子上一看，的确是能加上图标了，但是要卡几秒。。这是因为`data`的解析需要时间，阻塞了主线程。

于是，我便开多一个线程咯，反正开线程又不会阻塞主线程..
```
dispatch_group_async(group, queue, ^{
    //获取字节，用于判断动图
    NSData *data = [NSData dataWithContentsOfURL: [NSURL URLWithString: imageUrl]];
    [data getBytes:&c length:1];
    dispatch_async(dispatch_get_main_queue(), ^{
        if (c == 0x47 || isLong) {
            self.longImageLabel.hidden = NO;
            if (isLong) {
                //如果是长图
                self.longImageLabelText.text = @"长图";
            }
            if (c == 0x47) {
                //如果是GIF，即便是长GIF也会显示动图
                self.longImageLabelText.text = @"动图";
            }
        } else {
            self.longImageLabel.hidden = YES;
        }
    });
});
```
emmm，这样的确是不阻塞主线程了，但是有时候图动起来了都没加上相应的标签..

一怒之下看了这个解析GIF的代码究竟是何方神圣：
```
- (void)sd_setImageWithURL:(nullable NSURL *)url
          placeholderImage:(nullable UIImage *)placeholder
                   options:(SDWebImageOptions)options
                  progress:(nullable SDWebImageDownloaderProgressBlock)progressBlock
                 completed:(nullable SDExternalCompletionBlock)completedBlock {
    __weak typeof(self)weakSelf = self;
    [self sd_internalSetImageWithURL:url
                    placeholderImage:placeholder
                             options:options
                        operationKey:nil
                       setImageBlock:^(UIImage *image, NSData *imageData) {
                           SDImageFormat imageFormat = [NSData sd_imageFormatForImageData:imageData];
                           if (imageFormat == SDImageFormatGIF) {
                               weakSelf.animatedImage = [FLAnimatedImage animatedImageWithGIFData:imageData];
                               weakSelf.image = nil;
                           } else {
                               weakSelf.image = image;
                               weakSelf.animatedImage = nil;
                           }
                       }
                            progress:progressBlock
                           completed:completedBlock];
}
```
咦，里面已经有关于GIF的判断了，那其实我们直接在他判断为GIF的时候加上动图的标签就可以了，但是！！这是`FLAnimatedImageView`的内容，那当然就不可以直接改啦！这时候就要用到我们的类别了！
```
- (BOOL)cc_setImageWithURL:(nullable NSURL *)url
          placeholderImage:(nullable UIImage *)placeholder
                   options:(SDWebImageOptions)options
                  progress:(nullable SDWebImageDownloaderProgressBlock)progressBlock
                 completed:(nullable SDExternalCompletionBlock)completedBlock {
    __block BOOL isGIF = NO;
    __weak typeof(self)weakSelf = self;
    [self sd_internalSetImageWithURL:url
                    placeholderImage:placeholder
                             options:options
                        operationKey:nil
                       setImageBlock:^(UIImage *image, NSData *imageData) {
                           SDImageFormat imageFormat = [NSData sd_imageFormatForImageData:imageData];
                           if (imageFormat == SDImageFormatGIF) {
                               weakSelf.animatedImage = [FLAnimatedImage animatedImageWithGIFData:imageData];
                               weakSelf.image = nil;
                               isGIF = YES;
                           } else {
                               weakSelf.image = image;
                               weakSelf.animatedImage = nil;
                               isGIF = NO;
                           }
                       }
                            progress:progressBlock
                           completed:completedBlock];
    return isGIF;
}
```
没错，只要在类别中重写这个函数，然后再加上isGIF的返回类型，调用这个类别的方法，即可获取是否为GIF了！

---
###_更新于8.2_
以上方法虽然可以做到判断GIF，但是还是会有偶发现象：GIF标签有时候并不能及时地贴上

在debug过程中可以发现，在加载GIF时，`imageFormat`并不会立即被判断为`SDImageFormatGIF`，而是`SDImageFormatUndefined`，然后再看看源码，发现是因为`imageData`一开始被设置为`nil`，导致图片类型被判断为`SDImageFormatUndefined`了。

为什么Data会被设置为nil？

科普一下`options`参数的作用，来源在[这里](http://www.cnblogs.com/WJJ-Dream/p/5816750.html)：
```
typedef NS_OPTIONS(NSUInteger, SDWebImageOptions) {

    SDWebImageRetryFailed = 1 << 0,

    SDWebImageLowPriority = 1 << 1,

    SDWebImageCacheMemoryOnly = 1 << 2,

    SDWebImageProgressiveDownload = 1 << 3,

    SDWebImageRefreshCached = 1 << 4,

    SDWebImageContinueInBackground = 1 << 5,

    SDWebImageHandleCookies = 1 << 6,

    SDWebImageAllowInvalidSSLCertificates = 1 << 7,

    SDWebImageHighPriority = 1 << 8,

    SDWebImageDelayPlaceholder = 1 << 9,

    SDWebImageTransformAnimatedImage = 1 << 10,

    SDWebImageAvoidAutoSetImage = 1 << 11
};
```
SDWebImageRetryFailed = 1 << 0,:默认情况下,如果一个url在下载的时候失败了,那么这个url会被加入黑名单并且library不会尝试再次下载,这个flag会阻止library把失败的url加入黑名单(简单来说如果选择了这个flag,那么即使某个url下载失败了,sdwebimage还是会尝试再次下载他

SDWebImageLowPriority = 1 << 1,:默认情况下,图片会在交互发生的时候下载(例如你滑动tableview的时候),这个flag会禁止这个特性,导致的结果就是在scrollview减速的时候,才会开始下载(也就是你滑动的时候scrollview不下载,你手从屏幕上移走,scrollview开始减速的时候才会开始下载图片

SDWebImageCacheMemoryOnly = 1 << 2,:这个flag禁止磁盘缓存,只有内存缓存

SDWebImageProgressiveDownload = 1 << 3,:这个flag会在图片下载的时候就显示(就像你用浏览器浏览网页的时候那种图片下载,一截一截的显示(待确认))

SDWebImageRefreshCached = 1 << 4,:一个图片缓存了,还是会重新请求.并且缓存侧略依据NSURLCache而不是SDWebImage.URL不变,图片会更新时使用

SDWebImageContinueInBackground = 1 << 5,:启动后台下载,加入你进入一个页面,有一张图片正在下载这时候你让app进入后台,图片还是会继续下载(这个估计要开backgroundfetch才有用)

SDWebImageHandleCookies = 1 << 6,:可以控制存在NSHTTPCookieStore的cookies.

SDWebImageAllowInvalidSSLCertificates = 1 << 7,:允许不安全的SSL证书,在正式环境中慎用

SDWebImageHighPriority = 1 << 8,:默认情况下,image在装载的时候是按照他们在队列中的顺序装载的(就是先进先出).这个flag会把他们移动到队列的前端,并且立刻装载,而不是等到当前队列装载的时候再装载.

SDWebImageDelayPlaceholder = 1 << 9,:默认情况下,占位图会在图片下载的时候显示.这个flag开启会延迟占位图显示的时间,等到图片下载完成之后才会显示占位图.

SDWebImageTransformAnimatedImage = 1 << 10,:是否transform图片

而源码中有这样的语句：
```
if (!(options & SDWebImageDelayPlaceholder)) {
        dispatch_main_async_safe(^{
            [self sd_setImage:placeholder imageData:nil basedOnClassOrViaCustomSetImageBlock:setImageBlock];
        });
    }
```
可以看到，在显示占位图模式（不设置`SDWebImageDelayPlaceholder`）中，首先这个方法会先加载一张占位图，与此同时，原图（GIF等）中的data会先被设置为`nil`，导致这张图片先会被设置为`SDImageFormatUndefined`。

而在之前的方法中，如果不是GIF类型则直接返回`NO`，也导致了标签会直接不显示出来，等到真正加载GIF的时候，标签已成定局，也就是说，无法再为动图加上标签了。

所以，之前提到的`直接返回一个BOOL变量的方法`是不太准确的，最好的方法是使用`委托模式`或者`观察者模式`，委托或者通知。

但是这是一个类别啊喂，怎么拥有`delegate`变量？委托这条路显然是走不通的了。那就用通知吧。

我们知道，通知是那种一对多的关系，怎么让通知变成一对一呢？先来学习一下通知的几个变量：

[iOS 通知（NSNotification）的简单使用](https://blog.csdn.net/m0_37681833/article/details/68064700)中提到，NSNotification中，各种参数分别是：
```
name：是消息对象的唯一标识，接受通知消息时用来辨别

@property (readonly,copy)NSNotificationName name;

object：一个对象，可以理解为针对某个对象的消息

@property (nullable,readonly,retain)id object;

userInfo：一个字典，用来传值

@property (nullable,readonly,copy)NSDictionary *userInfo;
```
其中，`name`和`object`不设为`nil`的话就是一对一关系了。图片的URL对应的String即为唯一标识符。

哟西，思路已经很清晰了，当我们加载到一张GIF图片的时候，我们就会发出一个通知，让cell去显示这个动图的标签，否则，则要么隐藏标签，要么显示长图的标签。
```
SDImageFormat imageFormat = [NSData sd_imageFormatForImageData:imageData];
if (imageFormat == SDImageFormatGIF) {
   weakSelf.animatedImage = [FLAnimatedImage animatedImageWithGIFData:imageData];
   weakSelf.image = nil;
   NSNotification * notice = [NSNotification notificationWithName: imageUrl object: imageUrl userInfo: @{@"isGIF": @"GIF"}];
   [[NSNotificationCenter defaultCenter]postNotification:notice];
} else {
   weakSelf.image = image;
   weakSelf.animatedImage = nil;
   NSNotification * notice = [NSNotification notificationWithName: imageUrl object: imageUrl userInfo: @{@"isGIF": @"NotGIF"}];
   [[NSNotificationCenter defaultCenter]postNotification:notice];
}
```
同时，再为cell注册通知和响应方法
```
NSNotificationCenter *center = [NSNotificationCenter defaultCenter];
[center addObserver: self selector: @selector(setGIFLabel:) name: imageUrl object: imageUrl];
```
```
- (void)setGIFLabel: (NSNotification *)notification {
    BOOL isGIF = [notification.userInfo[@"isGIF"] isEqualToString: @"GIF"];
    if (isGIF || self.isLong) {
        self.longImageLabel.hidden = NO;
        if (isGIF)
            self.longImageLabelText.text = @"动图";
        else
            self.longImageLabelText.text = @"长图";
    } else {
        self.longImageLabel.hidden = YES;
    }
}
```
这样就可以很及时准确地将动图贴上自己的标签啦！

---
###_更新于8.3_
因为性能问题被QA指出来批评了..虽然通知不是很耗性能吧，但是一对一的通知，总感觉怪怪的..还是改成委托比较好。

之前我说过，类别不能用委托，所以这次我们改成继承吧。继承于基类ImageView，然后在Cell中改成自定义imageView即可。

最后声明协议和方法，在对应的情形下使用对应的协议即可。