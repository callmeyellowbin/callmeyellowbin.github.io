在做`View Controller`的生命周期的时候，由于要用到`UINavigationController`，就干脆继续学习了一波这个知识点，参考了[这篇文章](https://www.jianshu.com/p/b2ae4d211499)
###一.定义
`UINavigationController(导航控制器)`是一个容器控制器，其内部有多个`UIViewController(视图控制器)`的内容，我们可以通过`UINavigationController`的`view`属性获取到其自身的视图，在该视图上面有一个位于界面顶部的`UINavigationBar(导航栏)`和位于界面底部的默认隐藏的`UIToolbar(工具栏)`，以及一个位于界面中间部分的`UIViewController`的`view`。

![UINavigationController层级结构](http://upload-images.jianshu.io/upload_images/8407639-e39106724ee4aeec?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当用户在`UINavigationController`的层级结构中来回切换的时候，`UINavigationBar`和`UIToolbar`的内容会随之发生变化，但是其本身并不会发生变化，唯一发生变化的就是位于界面中间部分的`UIViewController`的`view`。
###二.UIViewController堆栈的管理
`UINavigationController`通过其管理的`UIViewController堆栈`来决定展示在`UINavigationController`中间部位的内容，该内容由位于`UIViewController堆栈`的栈顶位置的`UIViewController`决定。

下图的`View Controllers`是`UIViewController堆栈`，`navigationBar`是位于顶部的`UINavigationBar`，`toolBar`是位于底部的`UIToolbar`。

![UIViewController堆栈](http://upload-images.jianshu.io/upload_images/8407639-ee64083c9ccc306f?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

根据栈的定义，如果我们要实现这个栈的内容，既可以用push和pop对`UIViewController`进行操作，也可以直接设置`UIViewController堆栈`的内容。
####1.push
```
//push
//方法一
/*
 * 参数一: UIViewController, 该参数不可以使用UITabBarController的实例
 * 参数二: 是否执行动画
 */
[navigationController pushViewController:[[UIViewController alloc] init] animated:YES];

//方法二
/*
 * 参数一: UIViewController, 该参数不可以使用UITabBarController的实例
 * 参数二: 要求展示UIViewController的对象
 */
[navigationController showViewController:[[UIViewController alloc] init] sender:nil];
```
####2.pop
```
//pop一个UIViewController
/*
 * 参数一: 是否执行动画
 * 返回值: 从UIViewController堆栈中Pop出来的UIViewController
 */
UIViewController *viewController = [navigationController popViewControllerAnimated:YES];

//一直pop到根视图控制器
/*
 * 参数一: 是否执行动画
 * 返回值: 从UIViewController堆栈中Pop出来的UIViewController数组
 */
NSArray *viewControllers = [navigationController popToRootViewControllerAnimated:YES];

//一直pop到指定的UIViewController
/*
 * 参数一: 指定UIViewController, 该UIViewController必须位于当前UIViewController堆栈中
 * 参数二: 是否执行动画
 * 返回值: 从UIViewController堆栈中Pop出来的UIViewController数组
 */
NSArray *viewControllers = [navigationController popToViewController:navigationController.viewControllers[0] animated:YES];
```
####3.获取
```
// 获取位于UIViewController堆栈栈顶位置的UIViewController
UIViewController *viewController = navigationController.topViewController;
```
###三.UINavigationBar的管理
`UINavigationController`通过位于`UIViewController堆栈`栈顶位置的`UIViewController`的`navigationItem`属性(该属性位于`UIViewController`的`UINavigationControllerItem`类目中)来管理`UINavigationBar`展示的内容，同时`UINavigationController`也提供了`navigationBar`属性, 允许开发者通过该属性设置`UINavigationBar`的外观。

值得注意的是，`UINavigationController`是`UINavigationBar`的**delegate**, 其负责响应该`UINavigationBarDelegate`的代理方法, 并据此更新位于界面中间部分的`UIViewController`的视图。

####1.设置UINavigationBar的外观
我们可以通过该属性设置`UINavigationBar`的外观, 但是不要通过该属性设置其frame, bounds, alpha等属性, 更不要修改其层级结构。
```
// 属性
@property(nonatomic, readonly) UINavigationBar *navigationBar;
// 示例
navigationController.navigationBar.barStyle = UIBarStyleDefault;
```
####2.设置UINavigationBar的隐藏状态
```
// 属性
@property(nonatomic, getter=isNavigationBarHidden) BOOL navigationBarHidden;
// 示例
navigationController.navigationBarHidden = YES;
```
####3.设置UINavigationBar的隐藏状态(可选动画)
```
// 方法
/*
 * 参数一: 隐藏状态
 * 参数二: 是否执行动画
 */
- (void)setNavigationBarHidden:(BOOL)hidden animated:(BOOL)animated;
// 示例
[navigationController setNavigationBarHidden:YES animated:YES];
```
###四.UIToolbar的管理
`UINavigationController`通过位于`UIViewController堆栈`栈顶位置的`UIViewController`的`toolbarItems `属性(该属性位于`UIViewController`的`UINavigationControllerContextualToolbarItems `类目中)来管理`UIToolbar `展示的内容，同时`UINavigationController`也提供了`toolbar `属性, 允许开发者通过该属性设置`UIToolbar `的外观。
####1.设置UIToolbar的外观
```
// 属性
@property(nonatomic, readonly) UIToolbar *toolbar;
// 示例
navigationController.toolbar.barStyle = UIBarStyleDefault;
```
####2.设置UIToolbar的隐藏状态
```
// 属性
@property(nonatomic, getter=isToolbarHidden) BOOL toolbarHidden;
// 示例
navigationController.toolbarHidden = NO;
```
####3.设置UIToolbar的隐藏状态(可选动画)
```
// 方法
/*
 * 参数一: 隐藏状态
 * 参数二: 是否执行动画
 */
- (void)setToolbarHidden:(BOOL)hidden animated:(BOOL)animated;
// 示例
[navigationController setToolbarHidden:NO animated:YES];
```
###五.手势识别器的管理
####1.获取手势识别器
```
// 侧滑返回手势识别器
@property(nonatomic, readonly) UIGestureRecognizer *interactivePopGestureRecognizer;
// 用于轻拍隐藏UINavigationBar与UIToolbar的手势识别器
@property(nonatomic, readonly, assign) UITapGestureRecognizer *barHideOnTapGestureRecognizer;
// 用于轻扫隐藏UINavigationBar与UIToolbar的手势识别器
@property(nonatomic, readonly, strong) UIPanGestureRecognizer *barHideOnSwipeGestureRecognizer;
// 示例
UIGestureRecognizer *interactivePopGestureRecognizer = navigationController.interactivePopGestureRecognizer;
```
####2.通过手势隐藏UINavigationBar与UIToolbar
```
// 轻拍隐藏、再次轻拍显示
@property(nonatomic, readwrite, assign) BOOL hidesBarsOnTap;
// 向上轻扫隐藏、向下轻扫显示
@property(nonatomic, readwrite, assign) BOOL hidesBarsOnSwipe;
// 横屏隐藏(此时轻拍显示)、竖屏显示
@property(nonatomic, readwrite, assign) BOOL hidesBarsWhenVerticallyCompact;
// 键盘出现隐藏、键盘消失保持隐藏(此时轻拍显示)
@property(nonatomic, readwrite, assign) BOOL hidesBarsWhenKeyboardAppears;
// 示例
navigationController.hidesBarsOnTap = YES;
```
###六.UINavigationController对象的初始化
```
//通过UIViewController初始化
/*
 * 参数一: UIViewController, 该参数不可以使用UITabBarController的实例
 */
UINavigationController *navigationController = [[UINavigationController alloc] initWithRootViewController:[[UIViewController alloc] init]];

//通过UINavigationBar、UIToolbar初始化
/*
 * 参数一: 自定义UINavigationBar的子类, 如果是nil则为UINavigationBar类
 * 参数二: 自定义UIToolbar的子类, 如果是nil则为UIToolbar类
 */
UINavigationController *navigationController = [[UINavigationController alloc] initWithNavigationBarClass:[UINavigationBar class] toolbarClass:[UIToolbar class]];
```
###七. UINavigationBar
####1.概述
`UINavigationBar`是一个在层级结构中起导航作用的视觉控件, 其一般展示形式如下图所示

![](http://upload-images.jianshu.io/upload_images/8407639-01b05c4b68707682?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
####2.UINavigationItem堆栈的管理
`UINavigationBar`虽然继承自`UIView`, 但是其并非通过`addSubview:`方法来添加子视图, 而是通过其管理的`UINavigationItem堆栈`来决定展示在`UINavigationBar`中的内容。

如下图所示，其中：
`items`是`UINavigationItem堆栈`;
`topItem`是位于`UINavigationItem堆栈`栈顶位置的`UINavigationItem`; 
`backItem`是位于`UINavigationItem堆栈`栈顶下方位置的`UINavigationItem`。

![](http://upload-images.jianshu.io/upload_images/8407639-0ee0d66471919a64?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们可以利用系统提供的方法向`UINavigationItem堆栈`中Push一个`UINavigationItem`, 从`UINavigationItem堆栈`中Pop一个`UINavigationItem`, 也可以直接设置`UINavigationItem堆栈`中的全部`UINavigationItem`。
####3. UINavigationBar的内容
`UINavigationBar`通过`UINavigationItem堆栈`按照如下方式来决定展示在`UINavigationBa`r中的内容

位于中间的标题会根据下方顺序选择展示的内容：

- 如果topItem设置了标题视图(titleView属性), 则展示标题视图

- 如果topItem设置了标题文字(title属性), 则展示标题文字

- 如果以上都未设置, 则展示空白

位于右侧的按钮会根据下方顺序选择展示的内容：

- 如果topItem设置了右侧按钮(rightBarButtonItem属性), 则展示右侧按钮

- 如果以上都未设置, 则展示空白

位于左侧的按钮会根据下方顺序选择展示的内容：

- 如果topItem设置了左侧按钮(leftBarButtonItem属性), 则展示左侧按钮

- 如果backItem设置了返回按钮(backBarButtonItem属性), 则展示返回按钮

- 如果backItem设置了标题文字(title属性), 则展示利用标题文字封装的返回按钮

- 如果以上都未设置, 则展示利用文字"Back"封装的返回按钮（前提是`UINavigationItem堆栈`中有超过一个的`UINavigationItem`）

```
// 初始化UINavigationItem对象
/*
 * 参数一: 标题文字
 */
- (instancetype)initWithTitle:(NSString *)title;
// 示例
UINavigationItem *navigationItem = [[UINavigationItem alloc] initWithTitle:@"Owen"];

//设置标题文字
// 属性
@property(nonatomic, copy) NSString *title;
// 示例
navigationItem.title = @"Title";

//设置标题视图
// 属性
@property(nonatomic, strong) UIView *titleView;
// 示例
navigationItem.titleView = [UIButton buttonWithType:UIButtonTypeInfoLight];

//设置提示文字
// 属性
@property(nonatomic, copy) NSString *prompt;
// 示例
navigationItem.title = @"Prompt";

//设置返回按钮
// 属性
@property(nonatomic, strong) UIBarButtonItem *backBarButtonItem;
// 示例
UIBarButtonItem *barButtonItem = [[UIBarButtonItem alloc] initWithTitle:@"Owen" style:UIBarButtonItemStylePlain target:nil action:nil];
navigationItem.backBarButtonItem = barButtonItem;
```
####4.UINavigationBar的外观
`UINavigationBar`类中提供了大量属性/方法用于设置其外观, 我们可以设置其样式、背景颜色、色彩颜色、文字属性等

![](http://upload-images.jianshu.io/upload_images/8407639-6b74e05375e0c660?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

同样, 我们也可以设置其背景图片、阴影图片等

![](http://upload-images.jianshu.io/upload_images/8407639-e87c3de2812747cf?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
//设置样式
//该属性默认为UIBarStyleDefault(白底黑字), 可选项为UIBarStyleBlack(黑底白字)
navigationBar.barStyle = UIBarStyleDefault;

//设置透明性
navigationBar.translucent = YES;

//设置背景颜色
navigationBar.barTintColor = [UIColor orangeColor];

//设置色彩颜色
navigationBar.tintColor = [UIColor greenColor];

//设置标题文字属性
navigationBar.titleTextAttributes = @{NSFontAttributeName: [UIFont systemFontOfSize:30], NSForegroundColorAttributeName: [UIColor whiteColor]};

//设置/获取背景图片
/*
 * 参数一: 背景图片
 * 参数二: 默认为UIBarPositionAny(不指定), 可选项为UIBarPositionBottom(在容器下方), UIBarPositionTop(在容器上方), UIBarPositionTopAttached(在屏幕上方, 与容器平级)
 * 参数三: 可选项为UIBarMetricsDefault(竖屏, 横屏未设置也使用该效果), UIBarMetricsCompact(横屏), UIBarMetricsDefaultPrompt(拥有提示文字的竖屏, 横屏未设置也使用该效果), UIBarMetricsCompactPrompt(拥有提示文字的横屏)
 */
[navigationBar setBackgroundImage:[UIImage imageNamed:@""] forBarPosition:UIBarPositionTop barMetrics:UIBarMetricsDefault];

//设置阴影图片
navigationBar.shadowImage = [UIImage imageNamed:@""];

//设置返回按钮图片（需要同时设置）
navigationBar.backIndicatorImage = [UIImage imageNamed:@""];
navigationBar.backIndicatorTransitionMaskImage = [UIImage imageNamed:@""];
```