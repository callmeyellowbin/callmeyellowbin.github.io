---
layout:     post
title:      iOS中如何优雅地处理票圈中的九图
#subtitle:   学习对象初始化
date:      2018-07-28
author:     Hoben Wong
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - iOS
    - Objective-C
    - 项目合集
---
在社交软件发达的今天，朋友圈的出现让每个人都能及时分享自己的生活，分享每时每刻成为了票圈最吸引人的地方。当我们要分享我们的生活时，就需要发布照片了。

但是，当照片的数量是5张，7张或者8张的时候，这就会显得非常尴尬了（这也是为什么很少有人发这个数量的票圈）。

![](https://upload-images.jianshu.io/upload_images/8407639-463184c0b43a93dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/8407639-707944120b1bf40b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/8407639-f20cb7aedbe7bfe3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

作为强迫症，我相信99%的人都不会选择发这么尴尬的数量，要么宁愿少发点，要么就凑多几张。

现在，强迫症的福利来了！你将会看到这样的布局！这样的布局是不是即使遇上了五张或者七张这种尴尬的数量，也不会尴尬呢，嘻嘻~

![](https://upload-images.jianshu.io/upload_images/8407639-5aaa093276fc4b46.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![](https://upload-images.jianshu.io/upload_images/8407639-9b29d61bf813de1f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![](https://upload-images.jianshu.io/upload_images/8407639-549f0b3f4f57c993.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


下面来看看这个功能在iOS端是怎么实现的吧~

###一.设计思路
单个票圈的布局中，一个Controller包括了很多个Cell，比如用户昵称Cell、图片Cell、评论点赞Cell等等，今天最主要设计的是图片的InfoCell。

![](https://upload-images.jianshu.io/upload_images/8407639-5b81476dd6a6cae8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####二.各层设计
####1. InfoCell
先来讲讲`InfoCell`是怎么设计的，因为这一层是核心，决定了整个图片的大小、位置等等。

首先，`InfoCell`采用`UICollectionView`来设计，这是因为`UICollectionView`更易于调整整个`section`的大小、布局等等。

在设计思路来说，`InfoCell`最重要的是需要根据自身的大小去决定每一个图片的大小。

比如说五图里面，前两个和后三个大小就很不一样，这时候就需要将他们分为两个part来讨论，5=2+3，7=3+4，而其他情况则可以直接按照原来的大小来设置：
```
- (NSInteger)numberOfSectionsInCollectionView:(UICollectionView *)collectionView {
    if (self.colorArray.count == 5 || self.colorArray.count == 7)
        return 2;
    else
        return 1;
}

- (NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section {
    if (self.colorArray.count == 5) {
        if (section == 0)
            return 2;
        else
            return 3;
    }
    else if (self.colorArray.count == 7) {
        if (section == 0)
            return 3;
        else
            return 4;
    }
    else
        return self.colorArray.count;
}
```
设置完Cell的数量之后，记得按情况获得每一个Cell的内容，还是要分情况讨论：
```
- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath {
    HobenNinePhotoCell *cell = [collectionView dequeueReusableCellWithReuseIdentifier: HobenNinePhotoID forIndexPath: indexPath];
    
    NSInteger currentItem = 0;
    if (self.colorArray.count == 5)
        currentItem = 2 * indexPath.section + indexPath.item;
    else if (self.colorArray.count == 7)
        currentItem = 3 * indexPath.section + indexPath.item;
    else
        currentItem = indexPath.item;
    
    [cell setColor: [self.colorArray objectAtIndex: currentItem]];
    
    return cell;
}
```
好的，最关键的地方来了，如何获得Cell的高宽的问题，这里我采用的是根据自身屏幕的宽度，按照比例、间距等关系来等比获取，即：

每个Cell的高宽 = (屏幕宽度 - 间距宽度 * 间距数量) / (Cell的数量)

又由于5张和7张有两种size，因此，5张的前两张是一种高度，后三张又是另外一种高度，7张同理，即可得到以下关系：

(`kHobenNinePhotoViewSpacing`即每个Cell之间的间距，这里取了5.f)
```
+ (CGFloat) getImageWidthWithWidth:(CGFloat)width index:(NSInteger)i count:(NSInteger) cnt {
    CGFloat spacing = kHobenNinePhotoViewSpacing;
    if (cnt == 2) {
        return (width - spacing) / 2;
    } else if (cnt % 3 == 0 || cnt == 4) {
        return (width - 2 * spacing) / 3;
    } else if (cnt == 8) {
        return (width - 3 * spacing) / 4;
    } else if (cnt == 5) {
        if (i <= 1) {
            return (width - spacing) / 2;
        } else {
            return (width - 2 * spacing) / 3;
        }
    } else if (cnt == 7) {
        if (i <= 2) {
            return (width - 2 * spacing) / 3;
        } else {
            return (width - 3 * spacing) / 4;
        }
    } else
        return 200.f;
}
```
在设置size的函数直接调用这个函数获取高宽就OK了：
```
- (CGSize)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout *)collectionViewLayout sizeForItemAtIndexPath:(NSIndexPath *)indexPath {
    NSInteger currentItem = 0;
    if (self.colorArray.count == 5)
        currentItem = 2 * indexPath.section + indexPath.item;
    else if (self.colorArray.count == 7)
        currentItem = 3 * indexPath.section + indexPath.item;
    else
        currentItem = indexPath.item;

    CGFloat width = [HobenNinePhotoView getImageWidthWithWidth: self.frame.size.width index: currentItem count: self.colorArray.count];
    
    return CGSizeMake(width, width);
}
```
由于5张和7张的情况是分成了两个部分的，因此，还需要调用`UICollectionViewDelegateFlowLayout`的委托来设置两个部分之间的间距：
```
- (UIEdgeInsets)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout *)collectionViewLayout insetForSectionAtIndex:(NSInteger)section {
    return UIEdgeInsetsMake(0, 0, kHobenNinePhotoViewSpacing, 0);
}

- (CGFloat)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout *)collectionViewLayout minimumLineSpacingForSectionAtIndex:(NSInteger)section {
    return kHobenNinePhotoViewSpacing;
}
```
最后，初始化`InfoCell`中的`CollectionView`，记得注册什么的：
```
- (UICollectionView *)ninePhotoView {
    if (!_ninePhotoView) {
        _ninePhotoView = ({
            UICollectionViewFlowLayout *layout = [[UICollectionViewFlowLayout alloc] init];
            [layout setMinimumLineSpacing:2.f];
            [layout setMinimumInteritemSpacing:2.f];
            [layout setScrollDirection:UICollectionViewScrollDirectionVertical];
            [layout setSectionInset:UIEdgeInsetsZero];
            UICollectionView *view = [[UICollectionView alloc] initWithFrame: self.bounds
                                                        collectionViewLayout: layout];
            view.dataSource = self;
            view.delegate = self;
            view.alwaysBounceVertical = YES;
            view.scrollEnabled = NO;
            view.backgroundColor = [UIColor clearColor];
            [view registerClass: [HobenNinePhotoCell class] forCellWithReuseIdentifier: HobenNinePhotoID];
            view;
        });
    }
    return _ninePhotoView;
}
```
####2. PhotoCell
PhotoCell即票圈布局里面的每一个照片，这时候，其实只要适配InfoCell给他设置好的高宽，还有内容就OK了：
```
- (void)layoutSubviews {
    [self.gridView mas_remakeConstraints:^(MASConstraintMaker *make) {
        make.size.equalTo(self);
        make.top.equalTo(self);
        make.left.equalTo(self);
    }];
}

- (void) setColor:(UIColor *)color {
    self.gridView.backgroundColor = color;
}
```
####3.Controller
`Controller`里面包含`InfoCell`，是用`UITableView`来包含的，但是如何获得每一个`TableView`的高度是一个难题，因为每一种布局里面的高度都是不固定的，如果计算高度发生错误，就会导致整个布局出现bug！

先初始化，设置委托，注册一下：
```
- (UITableView *)infoView {
    if (!_infoView) {
        _infoView = ({
            UITableView *tableView = [[UITableView alloc] initWithFrame: CGRectMake(0, 0,
                                                                               self.view.frame.size.width,
                                                                               self.view.frame.size.height)];
            [tableView registerClass: [HobenInfoCell class] forCellReuseIdentifier: HobenInfoCellID];
            tableView.delegate = self;
            tableView.dataSource = self;
            tableView;
        });
    }
    return _infoView;
}
```
这里的TableView就不用像Cell里面的CollectionView一样要分区处理了，直接按照数组数量来搞定！
```
#pragma mark - <UITableViewDelegate>
- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView {
    return 1;
}

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
    return self.colorArrayArray.count;
}
```

好的，又是一个关键的地方，怎么获得高度？这是一个值得思考的地方，想想之前是怎么获得的？参考一下，就可以知道，由于：

高度=宽度

每个Cell的高度 = (屏幕宽度 - 间距宽度 * 间距数量) / (Cell的数量)

TableView高度 = 每行Cell的高度之和

因为5张和7张比较特殊，上面一层的高度和下面一层的高度是不一致的，因此还是要分别加一下，而其他情况则可以直接用行数来获得：

```
+ (CGFloat) heightWithWidth: (CGFloat) width colorArray: (NSArray *) colorArray {
    CGFloat imageHeight = 0.f;
    if(colorArray.count > 1) {
        if (colorArray.count == 2) {
            imageHeight = (width - kHobenNinePhotoViewSpacing) / 2;
        } else if (colorArray.count % 3 == 0) {
            NSInteger imageLine = colorArray.count / 3;
            imageHeight = imageLine * ((width - 2 * kHobenNinePhotoViewSpacing) / 3 + kHobenNinePhotoViewSpacing);
        } else if (colorArray.count == 4) {
            imageHeight = 2 * width / 3;
        } else if (colorArray.count == 5) {
            imageHeight = (width - kHobenNinePhotoViewSpacing) / 2 + (width - 2 * kHobenNinePhotoViewSpacing) / 3 + kHobenNinePhotoViewSpacing;
        } else if (colorArray.count == 7) {
            imageHeight = (width - 2 * kHobenNinePhotoViewSpacing) / 3 + (width - 3 * kHobenNinePhotoViewSpacing) / 4 + kHobenNinePhotoViewSpacing;
        } else if (colorArray.count == 8) {
            imageHeight = 2 * (width - 3 * kHobenNinePhotoViewSpacing) / 4 + kHobenNinePhotoViewSpacing;
        }
    }
    else {
        return 200.f;
    }
    return imageHeight;
}
```
在TableView高度确定中调用这个函数就OK啦！
```
- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath {
    CGFloat height = [HobenNinePhotoView heightWithWidth: self.view.frame.size.width - 20 colorArray: self.colorArrayArray[indexPath.item]];
    return height + 20.f;
}
```
####4.数据的传递
最后特别说一下数据的传递过程，在demo中，数据源的产生在`Controller`层，那么如何精确地传递给`PhotoCell`呢？跟着我的思路，来看看从数据的创建到数据的传递吧！

在Controller层先建立一坨坨颜色：
```
- (NSMutableArray *)colorArrayArray {
    if (!_colorArrayArray) {
        _colorArrayArray = ({
            NSMutableArray *arrays = [[NSMutableArray alloc] init];
            UIColor *black = [UIColor blackColor];
            UIColor *yellow = [UIColor yellowColor];
            UIColor *red = [UIColor redColor];
            UIColor *green = [UIColor greenColor];
            UIColor *gray = [UIColor grayColor];
            UIColor *orange = [UIColor orangeColor];
            UIColor *darkGray = [UIColor darkGrayColor];
            UIColor *brown = [UIColor brownColor];
            UIColor *blue = [UIColor blueColor];
            
            NSArray *totalColorArray = @[black, yellow, red, green, gray, orange, darkGray, brown, blue];
            
            for (int i = 1; i <= 9; i++) {
                NSArray *array = [totalColorArray subarrayWithRange: NSMakeRange(0, i)];
                [arrays addObject: array];
            }
            arrays;
        });
    };
    return _colorArrayArray;
}
```
没错，他叫colorArrayArray，因为他是数组的数组，即[[1], [1,2], [1, 2, 3]...]，在TableView里面传递给每一个Cell的时候，就取出其中一个数组，传递下去：
```
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    HobenInfoCell *cell = [tableView dequeueReusableCellWithIdentifier: HobenInfoCellID forIndexPath: indexPath];
    [cell setColorArray: self.colorArrayArray[indexPath.item]];
    return cell;
}
```
好的，`数组的数组`拆成了`一个个数组`，进入了`InfoCell`层，`InfoCell`再把得到的数组传给自己的`CollectionView`(记得`reload`)：
```
- (void)setColorArray:(NSArray *)colorArray {
    if (_colorArray != colorArray) {
        _colorArray = colorArray;
        [self.ninePhotoView reloadData];
    }
}
```

`CollectionView`获得了数组之后，就可以把它传给`PhotoCell`啦！
```
- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath {
    HobenNinePhotoCell *cell = [collectionView dequeueReusableCellWithReuseIdentifier: HobenNinePhotoID forIndexPath: indexPath];
    
    NSInteger currentItem = 0;
    if (self.colorArray.count == 5)
        currentItem = 2 * indexPath.section + indexPath.item;
    else if (self.colorArray.count == 7)
        currentItem = 3 * indexPath.section + indexPath.item;
    else
        currentItem = indexPath.item;
    
    [cell setColor: [self.colorArray objectAtIndex: currentItem]];
    
    return cell;
}
```
最后在PhotoCell中，根据获得的颜色，来设置自身的背景就大功告成啦！
```
- (void) setColor:(UIColor *)color {
    self.gridView.backgroundColor = color;
}
```
####5.效果图
![](https://upload-images.jianshu.io/upload_images/8407639-1b679b791fc6ceeb.gif?imageMogr2/auto-orient/strip)
####6.Demo地址
我的Demo地址在[这里](https://github.com/callmeyellowbin/NinePhotoDemo)，欢迎各位路过的爷给个star~ 嘻嘻
