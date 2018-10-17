---
layout:     post
title:      UITableView的学习
#subtitle:   学习对象初始化
date:      2018-06-12
author:     Hoben Wong
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - iOS
    - Objective-C
---
### 一.设置纯文字的UITableView
首先我们要在Main.storyboard中，选择TableView，并拖动到View Controller中，设置好相应的约束。

然后开始声明委托：
```
#import "ViewController.h"

@interface ViewController () <UITableViewDelegate, UITableViewDataSource>
@property (copy, nonatomic) NSArray *array;
@end
```
初始化我们要显示的数组：
```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    self.array = @[@"Hoben", @"Eric", @"Road", @"jp", @"WhoWhenHow", @"GuanGuan", @"CJ", @"Shit",
                   @"VJ", @"LS", @"HouGo", @"XiaoYu", @"PangPang", @"William", @"WB", @"HR",
                   @"Xinyi", @"Weixin", @"Shuhua", @"Xiaoyu2", @"Kay", @"JY"];
}
```
然后实现委托的方法：
```
//第一个参数tableView是引用，指向当前构建的表
- (nonnull UITableViewCell *)tableView:(nonnull UITableView *)tableView
                 cellForRowAtIndexPath:(nonnull NSIndexPath *)indexPath {
    //作为键使用，用来表示某个表单元。
    static NSString *simpleTableIdentifier = @"simpleTableIdentifier";
    
    //获得simpleTableIdentifier类型的可重用单元
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier: simpleTableIdentifier];
    
if (cell == nil) {
    //确保它是simpleTableIdentifier创建的
    cell = [[UITableViewCell alloc] initWithStyle: UITableViewCellStyleDefault
                                  reuseIdentifier: simpleTableIdentifier];
}
    cell.textLabel.text = self.array[indexPath.row];
    return cell;
}
```
```
- (NSInteger)tableView:(nonnull UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
        return [self.array count];
}
```
### 二.为TableView加上标题和图片
在这里，我们更改一下UITableViewCell的展示方法：
```
//    cell = [[UITableViewCell alloc] initWithStyle: UITableViewCellStyleDefault
//                                  reuseIdentifier: simpleTableIdentifier];
cell = [[UITableViewCell alloc] initWithStyle: UITableViewCellStyleSubtitle
                                  reuseIdentifier: simpleTableIdentifier];
```
设置一下他们的subTitle：
```
if (indexPath.row < 15)
     cell.detailTextLabel.text = @"男生";
else
     cell.detailTextLabel.text = @"女生";
```
以及选中的图片
```
//设置普通的image
UIImage *image = [UIImage imageNamed: @"star"];
cell.imageView.image = image;

//设置选中的image
UIImage *highlightedImage = [UIImage imageNamed: @"star2"];
cell.imageView.highlightedImage = highlightedImage;
```
### 三.设置行距
在indentationLevelForRowAtIndexPath中可以设置一下
```
- (NSInteger) tableView:(UITableView *)tableView indentationLevelForRowAtIndexPath:(NSIndexPath *)indexPath {
    return indexPath.row % 4;
}
```
![](https://upload-images.jianshu.io/upload_images/8407639-e9618302c13f9bde.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 四.处理行的选择
#### 1.让指定的行不能被选中
在willSelectRowAtIndexPath中，设置选定的行的相应讨论结果，如果是某一行则不能选中。
```
- (NSIndexPath *) tableView:(UITableView *)tableView willSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
    if (indexPath.row == 0)
        return nil;
    else
        return indexPath;
}
```
#### 2.选定行后的操作
在didSelectRowAtIndexPath中，获得相应的NSString结果，并用UIAlertController显示出来。
```
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
    NSString *rowValue = _array[indexPath.row];
    NSString *msg = [[NSString alloc] initWithFormat: @"You choosed %@", rowValue];
    UIAlertController *alert = [UIAlertController alertControllerWithTitle: @"Row selected"
                                                                   message: msg
                                                            preferredStyle: UIAlertControllerStyleAlert];
    UIAlertAction *action = [UIAlertAction actionWithTitle: @"Sure"
                                                     style: UIAlertActionStyleDefault
                                                   handler: nil];
    [alert addAction: action];
    [self presentViewController: alert
                       animated: YES
                     completion: nil];
}
```
### 五.更改字体大小和行高
在- (nonnull UITableViewCell *)tableView:(nonnull UITableView *)tableView cellForRowAtIndexPath:(nonnull NSIndexPath *)indexPath加上这一句
```
cell.textLabel.font = [UIFont boldSystemFontOfSize: 50];
```
然后再实现多一个方法
```
- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath
{
    return indexPath.row == 0 ? 120 : 70;
}
```
但是很丑哈哈哈哈哈哈哈
![](https://upload-images.jianshu.io/upload_images/8407639-e02394516fb9aece.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 六.创建自定义UITableViewCell
这次我们来重新定义一个cell，类似于Android里面的item项目。cell里面有两个属性：name和color，它们的文本内容以NSString形式存放在.h文件里面。
```
#import <UIKit/UIKit.h>

@interface NameAndColorCell : UITableViewCell

@property (copy, nonatomic) NSString *name;
@property (copy, nonatomic) NSString *color;
@end
```
然后在.m文件定义name和color对应的标签
```
#import "NameAndColorCell.h"
@interface NameAndColorCell ()
@property (strong, nonatomic) UILabel *nameLabel;
@property (strong, nonatomic) UILabel *colorLabel;

@end
```
这次尝试用代码形式来构建这个item：
```
- (instancetype)initWithStyle:(UITableViewCellStyle)style reuseIdentifier:(NSString *)reuseIdentifier
{
    self = [super initWithStyle: style reuseIdentifier: reuseIdentifier];
    if (self) {
        //初始化代码
        
        //静态文本 Name:
        //CGRect (0,5)开始，width=70，height=15的矩形
        CGRect nameLabelRect = CGRectMake(0, 5, 70, 15);
        UILabel *nameMarker = [[UILabel alloc] initWithFrame: nameLabelRect];
        nameMarker.textAlignment = NSTextAlignmentRight;
        nameMarker.font = [UIFont boldSystemFontOfSize: 12];
        nameMarker.text = @"Name:";
        [self.contentView addSubview: nameMarker];
        
        //静态文本 Color:
        CGRect colorLabelRect = CGRectMake(0, 26, 70, 15);
        UILabel *colorMarker = [[UILabel alloc] initWithFrame: colorLabelRect];
        colorMarker.textAlignment = NSTextAlignmentRight;
        colorMarker.text = @"Color:";
        colorMarker.font = [UIFont boldSystemFontOfSize: 12];
        [self.contentView addSubview: colorMarker];
        
        //动态文本 nameValue
        CGRect nameValueRect = CGRectMake(80, 5, 200, 15);
        self.nameLabel = [[UILabel alloc] initWithFrame: nameValueRect];
        [self.contentView addSubview: _nameLabel];
        
        //动态文本 colorValue
        CGRect colorValueRect = CGRectMake(80, 25, 200, 15);
        self.colorLabel = [[UILabel alloc] initWithFrame: colorValueRect];
        [self.contentView addSubview: _colorLabel];
    }
    return self;
}
```
将set方法也补全：
```
- (void)setName:(NSString *)name
{
    if (![name isEqualToString: _name]) {
        _name = [name copy];
        self.nameLabel.text = _name;
    }
}

- (void)setColor:(NSString *)color
{
    if (![color isEqualToString: _color]) {
        _color = [color copy];
        self.colorLabel.text = _color;
    }
}
```
在ViewController.m文件中，部署相应的cell以及里面的属性：
```
//
static NSString *CellTableIdentifier = @"CellTableIdentifier";


@interface ViewController ()
@property (weak, nonatomic) IBOutlet UITableView *tableView;
@property (strong, nonatomic) UITableView *viewTable;
//@property (copy, nonatomic) NSArray *computers;
@property (copy, nonatomic) NSArray *computers;

@end

```
表视图能够利用某种类注册机在需要时创建一个新单元。也就是说，如果注册了表视图用到的所有可重用标识符，就总是能访问到一个可用的表单元。
```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.

    self.computers = @[@{@"Name" : @"MacBook Air",  @"Color": @"Sliver"},
                       @{@"Name" : @"MacBook Pro",  @"Color": @"Sliver"},
                       @{@"Name" : @"iMac",  @"Color": @"Sliver"},
                       @{@"Name" : @"Mac Mini",  @"Color": @"Sliver"},
                       @{@"Name" : @"Mac Pro",  @"Color": @"Black"}];
    [self.tableView registerClass: [NameAndColorCell class]
           forCellReuseIdentifier: CellTableIdentifier];
}
```
当需要使用到已经注册的表视图时，则需要在构建cell的方法上面，获得已经注册好的可重用单元，并且将字典内的数据加载进去。
```
- (nonnull UITableViewCell *)tableView:(nonnull UITableView *)tableView cellForRowAtIndexPath:(nonnull NSIndexPath *)indexPath {
    //获得CellTableIdentifier类型的可重用单元
    NameAndColorCell *cell = [tableView dequeueReusableCellWithIdentifier: CellTableIdentifier];
    
    NSDictionary *rowData = self.computers[indexPath.row];
    cell.name = rowData[@"Name"];
    cell.color = rowData[@"Color"];
    return cell;
}
```
注意还要声明一下UITableView的cell的数量
```
- (NSInteger)tableView:(nonnull UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
    return [self.computers count];
}
```
![](https://upload-images.jianshu.io/upload_images/8407639-ed1ea04767e07ab0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 七.分组分区和索引分区
#### 1.实现控制器
我们用到一个字典，里面包含了A-Z开头的字典，字典里是对应字母开头的所有数组。

记得注册！！记得注册！！记得注册！！
```
#import "ViewController.h"

static NSString *SectionsTableIdentifier = @"SectionsTableIdentifier";

@interface ViewController ()

@property (copy, nonatomic) NSDictionary *names;
@property (copy, nonatomic) NSArray *keys;
@property (weak, nonatomic) IBOutlet UITableView *tableView;

@end
```
导入字典之后，按字典顺序来读取这个字典：
```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    
    [self.tableView registerClass: [UITableViewCell class]
           forCellReuseIdentifier: SectionsTableIdentifier];
    NSString *path = [[NSBundle mainBundle] pathForResource: @"sortednames"
                                                     ofType: @"plist"];
    self.names = [NSDictionary dictionaryWithContentsOfFile: path];
    self.keys = [[self.names allKeys] sortedArrayUsingSelector: @selector(compare:)];
    
}
```
这次由于是具有分组分区和索引分区，因此我们分别要实现索引（即处理字典）和分组（处理字典里面的数组）。
```
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier: SectionsTableIdentifier
                                                            forIndexPath: indexPath];
  
    //获得key
    NSString *key = self.keys[indexPath.section];
    
    //获得value
    NSArray *nameSection = self.names[key];
    cell.textLabel.text = nameSection[indexPath.row];
    
    return cell;
}
```
```
//Section的数量
- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView
{
    return [self.keys count];
}
```
```
//Section里面的数量
- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
{
    NSString *key = self.keys[section];
    NSArray *nameSection = self.names[key];
    return [nameSection count];
}
```
```
- (NSString *)tableView:(UITableView *)tableView titleForHeaderInSection:(NSInteger)section
{
    //获得索引
    return self.keys[section];
}
```
再在右方栏添加我们的索引栏
```
- (NSArray<NSString *> *)sectionIndexTitlesForTableView:(UITableView *)tableView
{
    return self.keys;
}
```
#### 2.学习debug
在我运行的时候，突然发现控制台报错了，数组越界，EXO me？而且控制台报错只会跳到main.m函数里面，这该咋办。。

后来咨询了霖哥，才发现Xcode里面要打一个异常断点才能跳转到具体出错的位置，如图所示。
![](https://upload-images.jianshu.io/upload_images/8407639-1585e16e05e53f62.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/8407639-8523f8b62b73f822.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
哦。。原来一不小心，把section和row搞混了，这里的section是字典的具体定位，即哪个字典。如果我不小心搞成了row，就直接用字典里面的索引值去找是哪个字典了，显然会越界。。
![](https://upload-images.jianshu.io/upload_images/8407639-adc5543b97eac242.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
解决了这个问题之后，就发现可以愉快地运行了！
![](https://upload-images.jianshu.io/upload_images/8407639-0b0bd2239dbcce90.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 八.添加搜索栏
#### 1.创建SearchResultController
新建一个文件，用于实现搜索栏：
```
#import <UIKit/UIKit.h>

@interface SearchResultController : UITableViewController <UISearchResultsUpdating>

- (instancetype)initWithNames: (NSDictionary *) names andKeys: (NSArray *) keys;

@end
```
在这里，它继承于UITableViewController，遵循UISearchResultsUpdating协议，这样就可以将其作为UISearchController类的委托。这样，用户在搜索栏输入的时候，协议中定义的某个方法就会被调用，以更新搜索结果。

SearchResultController需要访问主视图控制器显示的名字列表，因此，我们需要提供属性来保留用来在主视图控制器中显示的名字字典和关键字列表。

为此，我们需要定义几样东西：
1. 表单元的标识符
2. 搜索用到的字典和关键字列表
3. 对搜索结果数组的引用
```
#import "SearchResultController.h"
#import <Masonry.h>

static NSString * SectionsTableIdentifier = @"SectionsTableIdentifier";

@interface SearchResultController ()

@property (strong, nonatomic) NSDictionary *names;
@property (strong, nonatomic) NSArray *keys;
@property (strong, nonatomic) NSMutableArray *filteredNames;

@end
```
在这里，strong是因为我们只要保持对主视图控制器数据的引用，这样会比创建一个源数据副本高效很多。

接下来，则需要定义初始化函数：
```
- (instancetype)initWithNames:(NSDictionary *)names andKeys:(NSArray *)keys
{
    if (self = [super initWithStyle: UITableViewStylePlain]) {
        self.names = names;
        self.keys = keys;
        self.filteredNames = [[NSMutableArray alloc] init];
    }
    return self;
}
```
最后，记得在viewDidLoad方法中注册表单元的标识符：
```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    
    [self.tableView registerClass: [UITableViewCell class]
         forCellReuseIdentifier: SectionsTableIdentifier];
}
```
#### 2.调用SearchResultController
在ViewController.m文件中，在viewDidLoad方法新增初始化方法：
```
SearchResultController *resultsController = [[SearchResultController alloc] initWithNames: self.names
                                                                                  andKeys: self.keys];

//显示搜索结果时出现视图控制器
self.searchController = [[UISearchController alloc] initWithSearchResultsController: resultsController];

//获取并配置UISearchBar
UISearchBar *searchBar = self.searchController.searchBar;
searchBar.scopeButtonTitles = @[@"All", @"Short", @"Long"];
searchBar.placeholder = @"Enter a search item";

//将搜索栏关联到用户界面，并将其作为主视图控制器中表视图的顶部视图
[searchBar sizeToFit];
self.tableView.tableHeaderView = searchBar;

//为searchResultsUpdater赋值
self.searchController.searchResultsUpdater = resultsController;
```
用户每次在搜索栏输入时，UISearchController就会使用searchResultsUpdater属性的对象来更新搜索结果。这时候只需要在SearchResultController类中处理相应搜索就可以了。
#### 3.处理关键字
首先在updateSearchResultsForSearchController方法中，得到searchController对应的text，从而去遍历每个字典里面的关键词，进行匹配。
```
- (void)updateSearchResultsForSearchController:(UISearchController *)searchController
{
    NSString *searchString = searchController.searchBar.text;
    NSInteger buttonIndex = searchController.searchBar.selectedScopeButtonIndex;
    [self.filteredNames removeAllObjects];
    
    if (searchString.length > 0) {
        NSPredicate *predicate = [NSPredicate predicateWithBlock: ^BOOL(NSString *name, NSDictionary *b) {
            NSUInteger nameLength = name.length;
            if ((buttonIndex == shortNameButtonIndex && nameLength >= longNameSize) ||
                (buttonIndex == longNameButtonIndex && nameLength < longNameSize)) {
                return NO;
            }
            NSRange range = [name rangeOfString: searchString
                                        options: NSCaseInsensitiveSearch];
            //匹配是否成功
            return range.location != NSNotFound;
        }];
        for (NSString *key in self.keys) {
            //从每一个字典中筛选出符合predicate要求的名字，形成一个数组
            NSArray *matches = [self.names[key] filteredArrayUsingPredicate: predicate];
            [self.filteredNames addObjectsFromArray: matches];
        }
    }
     //记得加这一句，用于刷新搜索的TableView的显示
    [self.tableView reloadData];
}

```
加载对应的cell
```
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier: SectionsTableIdentifier
                                                            forIndexPath: indexPath];
    cell.textLabel.text = self.filteredNames[indexPath.row];
    return cell;
}
```
获得TableView的数量
```
- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
{
    return [self.filteredNames count];
}
```
最后为索引栏加上反色：
```
self.tableView.sectionIndexBackgroundColor = [UIColor blackColor];
self.tableView.sectionIndexTrackingBackgroundColor = [UIColor darkGrayColor];
self.tableView.sectionIndexColor = [UIColor whiteColor];
```
效果好像不太行。。书上的代码有点坑，就这样吧= =
![](https://upload-images.jianshu.io/upload_images/8407639-e8efd145681ea9f2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 九.踩过的坑
#### 1.NSInternalInconsistencyException
Simple Table[1829:126199] *** Terminating app due to uncaught exception 'NSInternalInconsistencyException', reason: 'UITableView (<UITableView: 0x7fd5cb81b800; frame = (0 0; 414 736); clipsToBounds = YES; autoresize = RM+BM; gestureRecognizers = <NSArray: 0x604000444260>; layer = <CALayer: 0x600000031160>; contentOffset: {0, -20}; contentSize: {414, 968}; adjustedContentInset: {20, 0, 0, 0}>) failed to obtain a cell from its dataSource (<ViewController: 0x7fd5c9e099b0>)'
是因为我没有初始化这个cell，导致cell没有空间。加上这段代码就OK了。
```
if (cell == nil) {
    //确保它是simpleTableIdentifier创建的
    cell = [[UITableViewCell alloc] initWithStyle: UITableViewCellStyleDefault
                                  reuseIdentifier: simpleTableIdentifier];
}
```
#### 2.NSInvalidArgumentException
 reason: 'must pass a class of kind UITableViewCell'
注册方法写错了，不小心把UITableViewCell写成了UITableView，这里改一下就可以了！
```
[self.tableView registerClass: [UITableViewCell class]
           forCellReuseIdentifier: SectionsTableIdentifier];
```
#### 3.NSInvalidArgumentException
是因为我把addObjectsFromArray看成了addObjects，导致加进去的是一个数组，最后传进一个字符串的时候就匹配失败了。
```
[self.filteredNames addObjectsFromArray: matches];
```
