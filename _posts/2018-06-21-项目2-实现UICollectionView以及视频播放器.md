---
layout:     post
title:      项目2-实现UICollectionView以及视频播放器
#subtitle:   学习对象初始化
date:      2018-06-21
author:     Hoben Wong
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - iOS
    - Objective-C
    - 项目合集
---
### 一.项目需求
![](https://upload-images.jianshu.io/upload_images/8407639-2b5ef211f0831c13.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 二.实现列表
本次列表展示参考博客为[ios - 用UICollectionView实现瀑布流详解](https://www.jianshu.com/p/2876bfe92df4)
具体分为Cell、Layout和Controller三个层面的实现，实现逻辑如下：
![](https://upload-images.jianshu.io/upload_images/8407639-72d6a305b35998f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 1.Cell
在Cell层，我们需要对其进行布局（用代码实现)，类似于Android里面设置weight一样，只不过我通过手动设置比例来设置它们布局的相对大小。
```
- (instancetype) initWithFrame:(CGRect)frame
{
    if (self = [super initWithFrame: frame]) {
        //设置imageView的布局
        _imageView  = [[UIImageView alloc] init];
        [self.contentView addSubview:_imageView];
        [_imageView mas_makeConstraints:^(MASConstraintMaker *make) {
            make.top.equalTo(self.contentView).offset(0);
            make.size.mas_equalTo(CGSizeMake(self.contentView.bounds.size.width, self.contentView.bounds.size.height * 0.85));
        }];
        //标记为需要重新布局
        [_imageView setNeedsLayout];
        
        //设置label的布局
        _titleLabel = [[UILabel alloc] init];
        [self.contentView addSubview:_titleLabel];
        _titleLabel.textAlignment = NSTextAlignmentCenter;
        _titleLabel.lineBreakMode = NSLineBreakByWordWrapping;
        _titleLabel.font = [UIFont systemFontOfSize: 13];
        [_titleLabel mas_makeConstraints:^(MASConstraintMaker *make) {
            make.top.equalTo(self.imageView.mas_bottom).offset(self.contentView.bounds.size.height * 0.04);
            make.bottom.equalTo(self.contentView.mas_bottom);
            make.size.mas_equalTo(CGSizeMake(self.contentView.bounds.size.width, self.contentView.bounds.size.height * 0.11));
        }];
    }
    return self;
}
```
有关setNeedsLayout的可以参考[setNeedsLayout与layoutIfNeeded的区别](https://www.jianshu.com/p/d46bcc656e04)

为了设置圆角，我们需要在layoutSublayersOfLayer中设置圆角，再用setNeedsLayout对UIImageView进行布局更新
```
- (void) layoutSublayersOfLayer:(CALayer *)layer
{
    //设置圆角
    UIBezierPath *maskPath;
    maskPath = [UIBezierPath bezierPathWithRoundedRect:self.bounds
                                     byRoundingCorners:(UIRectCornerTopLeft | UIRectCornerTopRight)
                                           cornerRadii:CGSizeMake(5.0f, 5.0f)];
    CAShapeLayer *maskLayer = [[CAShapeLayer alloc] init];
    maskLayer.frame = self.bounds;
    maskLayer.path = maskPath.CGPath;
    self.layer.mask = maskLayer;
}
```
#### 2.Layout
在Layout层，我们则需要对Cell的放置、高度等属性进行相应的设置，由于原demo是可以实现瀑布流的，其思路是：每次都将cell插入到瀑布流中最短的那一列，然后实时更新每一列的高度，直到cell放置结束。

但是我实现瀑布流之后发现效果惨不忍睹（因为给我的封面的高度和宽度参差不齐，不好看），因此套用了demo的模板对布局进行展示：

首先，通过Controller实现的委托，获得Cell对应的属性，将其放入属性数组当中：
```
//返回indexPath对应cell的布局属性
- (UICollectionViewLayoutAttributes *) layoutAttributesForItemAtIndexPath:(NSIndexPath *)indexPath
{
    //创建布局属性
    UICollectionViewLayoutAttributes *attrs = [UICollectionViewLayoutAttributes layoutAttributesForCellWithIndexPath: indexPath];
    
    //collectionView的宽度
    CGFloat collectionViewWidth = self.collectionView.frame.size.width;
    
    //设置布局属性的frame
    CGFloat cellWidth = (collectionViewWidth - self.edgeInsets.left - self.edgeInsets.right - (self.columnCount - 1) * self.columnMargin) / self.columnCount;
    CGFloat cellHeight = cellWidth * 0.8;
    
    //找出最短那一列
    NSInteger destColumn = 0;
    CGFloat minColumnHeight = [self.columnHeights[0] doubleValue];
    
    for (int i = 1; i < self.columnCount; i++) {
        //取得第i列的高度
        CGFloat columnHeight = [self.columnHeights[i] doubleValue];
        
        if (minColumnHeight > columnHeight) {
            minColumnHeight = columnHeight;
            destColumn = I;
        }
    }
    
    //设置cell的坐标
    CGFloat cellX = self.edgeInsets.left + destColumn * (cellWidth + self.columnMargin);
    CGFloat cellY = minColumnHeight;
    if (cellY != self.edgeInsets.top)
        cellY += self.rowMargin;
    
    attrs.frame = CGRectMake(cellX, cellY, cellWidth, cellHeight);
    
    //更新最短那一列的高度
    self.columnHeights[destColumn] = @(CGRectGetMaxY(attrs.frame));
    
    //记录内容的高度（即最长那一列的高度）
    CGFloat maxColumnHeight = [self.columnHeights[destColumn] doubleValue];
    if (self.contentHeight < maxColumnHeight)
        self.contentHeight = maxColumnHeight;
    
    return attrs;
}
```
获得属性数组之后，在prepareLayout中，对所有的属性（高宽、位置、页边距）对应的数组进行初始化
```
//初始化
- (void) prepareLayout
{
    [super prepareLayout];
    
    self.contentHeight = 0;
    
    //清除之前计算的所有高度
    [self.columnHeights removeAllObjects];
    
    // 设置每一列默认的高度
    for (NSInteger i = 0; i < HobenDefaultColumnCount ; i ++) {
        [self.columnHeights addObject:@(HobenDefaultEdgeInsets.top)];
    }
    
    // 清除之前所有的布局属性
    [self.attrsArr removeAllObjects];
    
    // 开始创建每一个cell对应的布局属性
    NSInteger count = [self.collectionView numberOfItemsInSection:0];
    
    for (int i = 0; i < count; i++) {
        // 创建位置
        NSIndexPath * indexPath = [NSIndexPath indexPathForItem:i inSection:0];
        
        // 获取indexPath位置上cell对应的布局属性
        UICollectionViewLayoutAttributes * attrs = [self layoutAttributesForItemAtIndexPath:indexPath];
        
        [self.attrsArr addObject:attrs];
    }
}
```
设置Cell的大小：
```
//cell的大小
- (NSArray<UICollectionViewLayoutAttributes *> *) layoutAttributesForElementsInRect:(CGRect)rect
{
    return self.attrsArr;
}
```
注意cell和cell之间是有间距的，需要通过计算，获得Layout大小：
```
//内容的大小
- (CGSize) collectionViewContentSize
{
    return CGSizeMake(0, self.contentHeight + self.edgeInsets.bottom);
}
```
#### 3.Controller
Controller是对每一个Cell的内容属性（封面图片等）进行设置，并且获得每一个Cell的布局属性，实现委托，传递给Layout：

首先，一定要记得，有个ID需要注册：
```
static NSString * const HobenCoverId = @"HobenCoverId";

/**
 * 创建布局和collectionView
 */
- (void)setupLayoutAndCollectionView
{
    
    // 创建布局
    HobenWaterFallLayout * waterFallLayout = [[HobenWaterFallLayout alloc] init];
    waterFallLayout.delegate = self;
    
    // 创建collectionView
    UICollectionView * collectionView = [[UICollectionView alloc] initWithFrame: self.view.bounds
                                                           collectionViewLayout: waterFallLayout];
    collectionView.backgroundColor = [UIColor whiteColor];
    
    //设置DataSource和Delegate
    collectionView.dataSource = self;
    collectionView.delegate = self;
    [self.view addSubview:collectionView];
    
    // 注册
    [collectionView registerClass: [HobenCoverCell class]
       forCellWithReuseIdentifier: HobenCoverId];
    
    self.collectionView = collectionView;
}
```

解析json：
 ```
- (void) refreshCover
{
    _isEnd = NO;
    [self.collectionView.mj_footer resetNoMoreData];
    NSURL *url = [NSURL URLWithString: @"http://*******"];
    
    NSString *jsonString;
    jsonString = [NSString stringWithContentsOfURL: url
                                          encoding: NSUTF8StringEncoding
                                             error: nil];
    
    NSData* jsonData;
    jsonData = [jsonString dataUsingEncoding: NSUTF8StringEncoding];
    
    //获得解析的json
    _dict = [NSJSONSerialization JSONObjectWithData: jsonData
                                            options: NSJSONReadingMutableContainers
                                              error: nil];
    
    //获得需要的json数组
    _totalCovers = _dict[@"data"][@"info_list"];
}
 ```
在本次项目中，我实现了加载，因此需要对获得的数据进行分页，在这里，我设置每页有10个数据：
```
/**
 * 初始化
 */
- (void)initialize
{
    [self refreshCover];
    //设置每页大小和当前页数
    _sectionNum = 10;
    _currentPageNum = 0;
    self.title = @"视频列表";
    self.view.backgroundColor = [UIColor whiteColor];
}
```
刷新控件相应的逻辑如下：
```
/**
* 刷新控件
*/
- (void)setupRefresh
{
    self.collectionView.mj_header = [MJRefreshNormalHeader headerWithRefreshingTarget:self refreshingAction:@selector(loadNewCovers)];
    self.collectionView.mj_header.backgroundColor = [UIColor whiteColor];
    [self.collectionView.mj_header beginRefreshing];
    
    self.collectionView.mj_footer = [MJRefreshAutoNormalFooter footerWithRefreshingTarget:self refreshingAction:@selector(loadMoreShops)];
    self.collectionView.mj_footer.backgroundColor = [UIColor whiteColor];
    self.collectionView.mj_footer.hidden = YES;
}
```
下拉刷新，将会加载新的数据，其实现逻辑如下：
```
/**
 * 加载新的视频
 */
- (void) loadNewCovers
{
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        [self refreshCover];
        
        //刷新，当前页数置0
        [_covers removeAllObjects];
        _currentPageNum = 0;
        int endValue = 0;
        
        //如果到底了
        if ((_currentPageNum + 1)* _sectionNum >= [_totalCovers count]) {
            endValue = (int)[_totalCovers count] - _currentPageNum * _sectionNum;
        }
        
        else {
            endValue = _sectionNum;
        }
        
        NSArray * cover = [_totalCovers subarrayWithRange: NSMakeRange(_currentPageNum * _sectionNum, endValue)];
        [_covers addObjectsFromArray:cover];
        
        // 刷新表格
        [self.collectionView reloadData];
        
        [self.collectionView.mj_header endRefreshing];
    });
}
```
上拉加载则需要判断是否到底，如果到底了，则需要显示no more data：
```
//加载更多视频
- (void) loadMoreCovers
{
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        if (_isEnd)
            return;
        
        //刷新
        _currentPageNum++;
        int endValue = 0;
        
        //如果到底了
        if ((_currentPageNum + 1)* _sectionNum >= [_totalCovers count]) {
            endValue = (int)[_totalCovers count] - _currentPageNum * _sectionNum;
            _isEnd = YES;
        }
        
        else {
            endValue = _sectionNum;
        }
        
        NSArray * cover = [_totalCovers subarrayWithRange: NSMakeRange(_currentPageNum * _sectionNum, endValue)];
        [_covers addObjectsFromArray:cover];
        
        // 刷新表格
        [self.collectionView reloadData];
        
        [self.collectionView.mj_footer endRefreshing];
        
        if (_isEnd)
            [self.collectionView.mj_footer endRefreshingWithNoMoreData];
    });
}
```
对每一个cell与数据一一对应，并设置进cell里面：
```
- (UICollectionViewCell *) collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath
{
    HobenCoverCell *cell = [collectionView dequeueReusableCellWithReuseIdentifier: HobenCoverId
                                                                     forIndexPath: indexPath];
    
    NSDictionary *cover = [[NSDictionary alloc] init];
    cover = self.covers[indexPath.item];
    
    /** 图片  */
    NSString *img = cover[@"cover"];
    
    /** 视频标题  */
    NSString *title = cover[@"title"];
    
    /** 视频字段  */
    NSString *flv = cover[@"flv"];
    /** 视频时长  */
    NSString *duration = cover[@"duration"];
    
    /** 列表到底  */
    NSString *end = cover[@"end"];
    
    /** 宽高  */
    NSNumber *height = cover[@"height"];
    NSNumber *width = cover[@"width"];
    
    HobenCover *cellCover = [[HobenCover alloc] init];
    
    //设置内容
    [cellCover setImg: img];
    [cellCover setTitle: title];
    [cellCover setFlv: flv];
    [cellCover setDuration: duration];
    [cellCover setEnd: end];
    [cellCover setW: [width floatValue]];
    [cellCover setH: [height floatValue]];
    
    cell.cover = cellCover;
    return cell;
}
```
设置点击跳转事件：
```
- (void) collectionView:(UICollectionView *)collectionView didSelectItemAtIndexPath:(NSIndexPath *)indexPath
{
    HobenCoverCell *cell = (HobenCoverCell *)[collectionView cellForItemAtIndexPath: indexPath];
    
    HobenCover *cover = cell.cover;
    
    
    HobenVideoController *videoController = [[HobenVideoController alloc] init];
    
    [videoController setCover: cover];
    
    //点击跳转
    [self.navigationController pushViewController: videoController
                                         animated: YES];
    
}
```
其他的就不用说了，和之前学习的TableView差不多，需要设置section数量和section里面的item的数量：
```
- (NSInteger) numberOfSectionsInCollectionView:(UICollectionView *)collectionView
{
    return 1;
}

- (NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section
{
    self.collectionView.mj_footer.hidden = self.covers.count == 0;
    return self.covers.count;
}
```
最后实现委托，传递给Layout：
```
- (CGFloat) waterFallLayout:(HobenWaterFallLayout *)waterFallLayout heightForItemAtIndexPath:(NSUInteger)indexPath itemWidth:(CGFloat)itemWidth
{
    
    NSDictionary *cover = _covers[indexPath];
    NSNumber *height = cover[@"height"];
    NSNumber *width = cover[@"width"];
    return itemWidth / [width floatValue] * [height floatValue];
}

- (CGFloat) rowMarginInWaterFallLayout:(HobenWaterFallLayout *)waterFallLayout
{
    return 10;
}

- (NSUInteger) columnCountInWaterFallLayout:(HobenWaterFallLayout *)waterFallLayout
{
    return 2;
}

- (UIEdgeInsets) edgeInsetdInWaterFallLayout:(HobenWaterFallLayout *)waterFallLayout
{
    return UIEdgeInsetsMake(10, 10, 10, 10);
}
```
### 三.实现视频播放
这次的视频播放器使用的是基于AVPlayer封装的ZFPlayer，不得不说这个作者真的很强大：
具体参考他的[GitHub](https://github.com/renzifeng/ZFPlayer)和[博客文档](https://www.jianshu.com/p/90e55deb4d51)，配置的话GitHub有说得很完整了。

当控制器的view将要布局子控件时，就会调用viewWillLayoutSubviews，因此我们首先对视频的布局进行配置：
```
- (void) viewWillLayoutSubviews
{
    [super viewWillLayoutSubviews];
    
    CGFloat w = CGRectGetWidth(self.view.frame);
    CGFloat h = w * 9 / 16;
    
    //视频布局
    CGFloat x = 0;
    CGFloat y = (CGRectGetHeight(self.view.frame) - h) / 2;
    self.containerView.frame = CGRectMake(x, y, w, h);
    
    w = 44;
    h = w;
    
    //按钮居中
    x = (CGRectGetWidth(self.containerView.frame) - w) / 2;
    y = (CGRectGetHeight(self.containerView.frame) - h) / 2;
    self.playBtn.frame = CGRectMake(x, y, w, h);
}
```
对于布局加载的各个函数的调用顺序，可以看[这篇文章](https://www.jianshu.com/p/fcfbd4919b0b)
而在控制器的view布局子控件完成时，将调用viewDidLayoutSubviews，在下面进行播放器的初始化：
```
- (void) initialize
{
    //将flv转换成mp4
    NSMutableString *flv = [[NSMutableString alloc] initWithString: self.cover.flv];
    if (flv == nil)
        return;
    NSRange range = [flv rangeOfString: @".flv"];
    [flv replaceCharactersInRange: range withString: @".mp4"];
    _flvReadOnly = [[NSString alloc] initWithString: flv];
    
    //设置相应布局
    self.view.backgroundColor = [UIColor whiteColor];
    [self.view addSubview:self.containerView];
    [self.containerView addSubview:self.playBtn];
    
    ZFAVPlayerManager *playerManager = [[ZFAVPlayerManager alloc] init];
    /// 播放器相关
    self.player = [ZFPlayerController playerWithPlayerManager: playerManager
                                                containerView: self.containerView];
    self.player.controlView = self.controlView;

    //全屏之后，顶部栏隐藏
    @weakify(self)
    self.player.orientationWillChange = ^(ZFPlayerController * _Nonnull player, BOOL isFullScreen) {
        @strongify(self)
        [self setNeedsStatusBarAppearanceUpdate];
    };
    
    //结束播放之后停止
    self.player.playerDidToEnd = ^(id  _Nonnull asset) {
        @strongify(self)
        if (self.player.isFullScreen) {
            [self.player enterFullScreen:NO animated:YES];
            dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(self.player.orientationObserver.duration * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
                [self.player stop];
            });
        } else {
            [self.player stop];
        }
    };
    
}
```
在播放键点击之后进行播放：
```
- (void) playClick:(UIButton *)sender
{
    self.player.assetURL = [NSURL URLWithString: _flvReadOnly];
    [self.player playTheIndex: 0];
    [self.controlView showTitle: self.cover.title
                 coverURLString: self.cover.img
                 fullScreenMode: ZFFullScreenModeLandscape];
}
```
在加载视频之前显示封面图，注意要先将封面图进行比例压缩：
```
- (UIImage*)imageCompressWithSimple:(UIImage*)image scaledToSize:(CGSize)size
{
    UIGraphicsBeginImageContext(size);
    [image drawInRect:CGRectMake(0,0,size.width,size.height)];
    UIImage* newImage = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return newImage;
}
```
获得压缩完毕的封面图后，我们可以用BackgroundColor的方法，让视频未加载前显示封面图：
```
- (UIView *) containerView
{
    if (!_containerView) {
        _containerView = [UIView new];
        
        NSURL *imageURL = [NSURL URLWithString: self.cover.img];
        NSData *data = [NSData dataWithContentsOfURL: imageURL];
        UIImage *image = [[UIImage alloc] initWithData: data];
        
        CGFloat w = CGRectGetWidth(self.view.frame);
        CGFloat h = w * 9 / 16;
        
        //设置封面图的大小
        image = [self imageCompressWithSimple: image scaledToSize: CGSizeMake(w, h)];
        UIColor *bgColor = [UIColor colorWithPatternImage: image];

        [_containerView setBackgroundColor: bgColor];
    }
    return _containerView;
}
```
对播放按钮进行初始化：
```
- (UIButton *) playBtn
{
    if (!_playBtn) {
        _playBtn = [UIButton buttonWithType:UIButtonTypeCustom];
        [_playBtn setImage:[UIImage imageNamed:@"播放"] forState:UIControlStateNormal];
        [_playBtn addTarget:self action:@selector(playClick:) forControlEvents:UIControlEventTouchUpInside];
    }
    return _playBtn;
}
```
大功告成，这个ZFPlayer支持手势滑动调节亮度、音量、进度、重力感应等功能，可以说是非常强大了。

还有一点就是，使用视频播放器的时候不要加断点！否则会出现一些很奇怪的错误，我是看了[这篇博客](https://blog.csdn.net/ProgrammerWorking/article/details/786107210)才知道的，真的太坑了。
### 四.问题反馈与纠正
#### 1.关于ViewDidLoad、viewDidLayoutSubviews、layoutSubviews的调用问题
参考文章：[UI篇－VC的生命周期以及UIView的layoutSubviews和drawRect方法](https://www.jianshu.com/p/f2abd7a6f572)
首先看看单个viewController的生命周期：
1. loadView：加载view 会多次调用并且会使viewWillLayoutSubviews、viewDidLayoutSubviews不再执行
2. **viewDidLoad**：view加载完毕
3. viewWillAppear：控制器的view将要显示
4. **viewWillLayoutSubviews**：控制器的view将要布局子控件
5. **viewDidLayoutSubviews**：控制器的view布局子控件完成
  这期间系统可能会多次调用viewWillLayoutSubviews 、    viewDidLayoutSubviews 俩个方法
6. viewDidAppear:控制器的view完全显示
7. viewWillDisappear：控制器的view即将消失的时候
8. viewDidDisappear：控制器的view完全消失的时候

整个控制器生命周期： **viewDidLoad** -> viewWillAppear -> **viewWillLayoutSubviews** -> **viewDidLayoutSubviews** -> viewDidAppear -> viewWillDisappear -> viewDidDisappear

**viewWillLayoutSubviews** 在 viewWillAppear 之后 viewDidAppear 之前执行，这个方法会被调用多次，如果在此创建视图，**可能会创建多个**，而且这个方法中执行耗时操作依然会造成跳转卡顿的问题。

**viewDidLoad**是当程序第一次加载view时调用，**以后都不会用到**，而**viewDidAppear**是每当切换到view时就调用。

科普完以上知识之后，再看看我的代码：
#####1) viewDidLoad和viewDidLayoutSubviews
```
//不正确的方法
- (void)viewDidLoad
{
    [super viewDidLoad];
}


- (void)viewDidLayoutSubviews
{
    [super viewDidLayoutSubviews];
    
    [self initialize];
}
```
可以看到，如果创建多个视图的话，就会不断加载，可以说会非常消耗了！
##### 2) viewDidLayoutSubviews和viewWillLayoutSubviews
再来看看我的布局代码放哪了：
```
- (void) viewWillLayoutSubviews
{
    [super viewWillLayoutSubviews];
    
    CGFloat w = CGRectGetWidth(self.view.frame);
    CGFloat h = w * 9 / 16;
    
    //视频布局
    CGFloat x = 0;
    CGFloat y = (CGRectGetHeight(self.view.frame) - h) / 2;
    self.containerView.frame = CGRectMake(x, y, w, h);
    
    w = 44;
    h = w;
    
    //按钮居中
    x = (CGRectGetWidth(self.containerView.frame) - w) / 2;
    y = (CGRectGetHeight(self.containerView.frame) - h) / 2;
    self.playBtn.frame = CGRectMake(x, y, w, h);
}
```
是的，放在了控制器的view**将要**布局子控件里面，还用到了view的frame！如果我view大小改变了的话，这个布局就会不准确了！
#### 2.关于使用frame进行布局的问题
在Cell的`initWithFrame`里面，我的布局是这样的：
```
[_imageView mas_makeConstraints:^(MASConstraintMaker *make) {
    make.top.equalTo(self.contentView).offset(0);
    make.size.mas_equalTo(CGSizeMake(self.contentView.bounds.size.width, self.contentView.bounds.size.height * 0.85));
}];
```
这样做的问题在于，我是直接获得了`contentView`的大小（即写死了），当其布局大小发生改变的时候，用`mas_equal`的方法可不会随之而改变。怎么办？改成这样：
```
[_imageView mas_makeConstraints:^(MASConstraintMaker *make) {
    make.top.equalTo(self.contentView).offset(0);
    make.width.equalTo(self.contentView);
    make.height.equalTo(self.contentView).multipliedBy(0.85);
}];
```
但因为offset的问题（无法设置成weight形式），imageView和Label之间还是会有约束冲突，这时候我们可以定义一个空白的View来解决：
```
//添加空隙
UIView *space = [[UIView alloc] init];
[self.contentView addSubview: space];
[space mas_makeConstraints:^(MASConstraintMaker *make) {
    make.width.equalTo(self.contentView);
    make.height.equalTo(self.contentView).multipliedBy(0.04);
}];
```
这样就可以按照0.85 : 0.04 : 0.11的比例来控制这个Cell布局了。
#### 3.关于网络请求与阻塞主线程的问题
在请求数据的时候，我曾经是这样请求的：
```
- (void) refreshCover
{
   _isEnd = NO;
   [self.collectionView.mj_footer resetNoMoreData];
   NSURL *url = [NSURL URLWithString: @"http://*******"];
   
   NSString *jsonString;
   jsonString = [NSString stringWithContentsOfURL: url
                                         encoding: NSUTF8StringEncoding
                                            error: nil];
   
   NSData* jsonData;
   jsonData = [jsonString dataUsingEncoding: NSUTF8StringEncoding];
   
   //获得解析的json
   _dict = [NSJSONSerialization JSONObjectWithData: jsonData
                                           options: NSJSONReadingMutableContainers
                                             error: nil];
   
   //获得需要的json数组
   _totalCovers = _dict[@"data"][@"info_list"];
}
```
这个有什么问题呢？问题在于，这个函数是写在了主线程里面，如果加载很久的话，就会造成阻塞UI的后果。幸运的是，AFNetWorking提供了异步请求方法，参考[iOS9之后AFNetWorking的使用(详细)](https://www.jianshu.com/p/ab246881efa9)，就可以将代码修改成这样：
```
NSString *URLString = @"http://*******";

//使用AFNetWorking请求
AFHTTPSessionManager *session = [AFHTTPSessionManager manager];

//这一行是解决bug的
session.responseSerializer.acceptableContentTypes=[NSSet setWithObjects:@"application/json", @"text/json", @"text/javascript",@"text/html", @"application/javascript", nil];

//get请求
[session GET: URLString
  parameters: nil
    progress: nil
     success: ^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
        NSLog(@"请求成功");
         
         //获得字典
        _dict = responseObject;
         
        //获得需要的json数组
        _totalCovers = _dict[@"data"][@"info_list"];
     }
     failure: ^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
         NSLog(@"请求失败:%@", error);
     }];
```
看到那行解决bug没有，那行好像是因为这个框架本身的问题导致报错`"Request failed: unacceptable content-type: application/javascript" `，参考了[解决方案](http://www.bubuko.com/infodetail-1226998.html)。

同样地，在cell里面，设置加载中的图片时，我也直接暴力地请求了网络：
```
NSURL *url = [NSURL URLWithString: @"https://image.baidu.com/search/detail?ct=503316480&z=0&ipn=d&word=正在加载图片gif&hs=2&pn=9&spn=0&di=122397146850&pi=0&rn=1&tn=baiduimagedetail&is=0%2C0&ie=utf-8&oe=utf-8&cl=2&lm=-1&cs=4034065388%2C2568359934&os=124701788%2C3310077975&simid=0%2C0&adpicid=0&lpn=0&ln=30&fr=ala&fm=&sme=&cg=&bdtype=0&oriquery=正在加载图片gif&objurl=http%3A%2F%2Fimgsrc.baidu.com%2Fforum%2Fw%3D580%2Fsign%3D4fc40444dec451daf6f60ce386fd52a5%2Faef6d933c895d143f95a783970f082025aaf0749.jpg&fromurl=ippr_z2C%24qAzdH3FAzdH3Fptjkwv_z%26e3Bkwt17_z%26e3Bv54AzdH3FrAzdH3Fndb8c999ad%3Frt1%3Dc0al99dn8n0%26fjj_sz%3D8&gsm=0&islist=&querylist="];
NSData *data = [NSData dataWithContentsOfURL: url];
    
 UIImage *image = [UIImage sd_animatedGIFWithData:data];
```
事实上，只需要将这个Gif下载下来，放入Assets文件里面，再这样加载就OK了：
```
UIImage *image = [UIImage imageNamed: @"loading"];
```
(解决失败，不知道怎么加载gif类型的Placeholder，算了。。)
#### 4.使用官方的UICollectionViewFlowLayout
鉴于自定义Layout太复杂，在这里还是尝试一下使用官方的`UICollectionViewFlowLayout`，好像真的免去了委托等繁杂的工作（不过还是需要计算，因为`UICollectionViewFlowLayout`好像没有提供列数这个接口）
```
//collectionView的宽度
CGFloat collectionViewWidth = self.view.frame.size.width;

//内边距
UIEdgeInsets edgeInsets = UIEdgeInsetsMake(10, 10, 10, 10);

//列数
int columnCount = 2;

//列间距
CGFloat columnMargin = 10;

//设置布局属性的frame
CGFloat cellWidth = (collectionViewWidth - edgeInsets.left - edgeInsets.right - (columnCount - 1) * columnMargin) / columnCount;
CGFloat cellHeight = cellWidth * 0.8;

UICollectionViewFlowLayout *layout = [[UICollectionViewFlowLayout alloc]init];

//设置UICollectionViewFlowLayout的属性
layout.scrollDirection = UICollectionViewScrollDirectionVertical;
layout.sectionInset = edgeInsets;
layout.itemSize = CGSizeMake(cellWidth, cellHeight);

// 创建collectionView
UICollectionView * collectionView = [[UICollectionView alloc] initWithFrame: self.view.bounds
                                                       collectionViewLayout: layout];
```
#### 5.分页加载与超时功能的完善
项目给出的HTTP的地址里面有page=和size=，这其实是用于分页加载的（之前一直理解错了= =）
所以我们要用占位符来得出加载出来的地址。（这里略）
同时，使用了`AFNetworking`进行了异步加载，则需要处理好`MJRefresh控件`和`AFNetworking`逻辑，`MJRefresh控件`的加载完成必须是在`AFNetworking`**请求完成并且读取完毕**之后才能隐藏：
```
//get请求
    [session GET: URLString
      parameters: nil
        progress: nil
         success: ^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
            NSLog(@"请求成功");
             
             //获得字典
            _dict = responseObject;
             
            //获得需要的json数组
            _totalCovers = _dict[@"data"][@"info_list"];
             //获得该数据请求是否结束
             if ([_dict[@"data"][@"end"] intValue] == 1)
                 _isEnd = YES;
             else
                 _isEnd = NO;
             if ([load isEqualToString: @"loadNewCovers"])  {
                 //加载成功，更新列表
                 [_covers addObjectsFromArray: _totalCovers];
                 [self.collectionView reloadData];
                 [self.collectionView.mj_header endRefreshing];
             }
             if ([load isEqualToString: @"loadMoreCovers"])  {
                 if (_isEnd) {
                     //加载到底，结束加载
                     [self.collectionView.mj_footer endRefreshingWithNoMoreData];
                     return;
                 }
                 else {
                     //加载成功，更新列表
                     [_covers addObjectsFromArray: _totalCovers];
                     [self.collectionView reloadData];
                     [self.collectionView.mj_footer endRefreshing];
                     
                 }
             }
         }
```
设置请求的超时时间：
```
[session.requestSerializer willChangeValueForKey: @"timeoutInterval"];
session.requestSerializer.timeoutInterval = 10.0f;
[session.requestSerializer didChangeValueForKey: @"timeoutInterval"];
```
在网络请求失败的同时，也需要抛出警告视图。
```
 failure: ^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
             //加载失败，抛出异常
             NSLog(@"请求失败:%@", error);
             
             UIAlertController *alert = [UIAlertController alertControllerWithTitle: @"加载超时"
                                                                            message: @"请检查你的网络"
                                                                     preferredStyle: UIAlertControllerStyleAlert];
             UIAlertAction *action = [UIAlertAction actionWithTitle: @"了解"
                                                              style: UIAlertActionStyleDefault
                                                            handler: nil];
             [alert addAction: action];
             [self presentViewController: alert
                                animated: YES
                              completion: nil];
             
             [self.collectionView.mj_header endRefreshing];
             [self.collectionView.mj_footer endRefreshing];
         }];
}
```
为防止下拉加载的时候多次误触控件，需要设置一个延时：
```
//加载更多视频
- (void) loadMoreCovers
{
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(0.5 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        //刷新
        _currentPageNum++;
        
        //开始加载
        [self refreshCover: @"loadMoreCovers"];
    });
}
```
自此，分页加载即可完成了！
#### 6.模拟差网络环境下的刷新
在[这里](https://stackoverflow.com/questions/39793871/network-link-conditioner-not-working-on-macos-sierra)下载网络环境模拟器`Network Link Conditioner` ：

一开始我的`footer`在差网络环境下不断上拉就会不断刷新`currentPageNum`，导致UI操作和刷新操作不同步，在这里需要添加一个逻辑：即在进行UI上拉操作后，`footer`应该在`UICollectionView`加载完成之后，才能继续上拉，由此，我加上了这样一个操作：`dispatch_async(dispatch_get_main_queue())`
```
 if ([load isEqualToString: @"loadNewCovers"])  {
     //加载成功，更新列表
     [_covers addObjectsFromArray: _totalCovers];
     [self.collectionView reloadData];
     dispatch_async(dispatch_get_main_queue(), ^{
         [self.collectionView.mj_header endRefreshing];
     });
 }
 if ([load isEqualToString: @"loadMoreCovers"])  {
     if (_isEnd) {
         //加载到底，结束加载
         [self.collectionView.mj_footer endRefreshingWithNoMoreData];
         return;
     }
     else {
         //加载成功，更新列表
         [_covers addObjectsFromArray: _totalCovers];
         [self.collectionView reloadData];
         
         dispatch_async(dispatch_get_main_queue(), ^{
             [self.collectionView.mj_footer endRefreshing];
         });
         
     }
 }
```
加上以后，即可实现我想要的逻辑。同时记得下拉刷新后`resetNoMoreData`:
```objective-c
if ([load isEqualToString: @"loadNewCovers"])  {
    [_covers removeAllObjects];
    [self.collectionView.mj_footer resetNoMoreData];
}
```
