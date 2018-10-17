---
layout:     post
title:      学习程序的执行顺序和UIViewController的生命周期
#subtitle:   学习对象初始化
date:      2018-07-11 
author:     Hoben Wong
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - iOS
    - Objective-C
---
参考自[这篇文章](https://www.cnblogs.com/junhuawang/p/5742535.html)和[这篇对前文的整合](https://www.jianshu.com/p/d60b388b19f5)
###一.程序启动执行顺序
####1.程序的入口
在`main`函数，设置`AppDelegate`为函数的代理。
####2.程序完成加载
`  -[AppDelegate application:didFinishLaunchingWithOptions:]`
####3.创建Window窗口
####4.程序被激活
`-[AppDelegate applicationDidBecomeActive:]`
####5.点击Home键
首先，程序会取消激活状态：`-[AppDelegate applicationWillResignActive:]`

然后，程序进入后台：`-[AppDelegate applicationDidEnterBackground:]`
####6.重新点击进入程序
首先，程序进入前台：`-[AppDelegate applicationWillEnterForeground:]`

然后，程序会被激活：`-[AppDelegate applicationDidBecomeActive:]`

与Android的生命周期有着异曲同工之妙，但也有些许区别，比如创建进程之后，是加载$\rightarrow$激活的，没有`applicationWillEnterForeground`：
1. 创建进程

onCreate() / applicationDidFinishLaunching

2. 进入视野（没获取焦点）

onStart() / applicationWillEnterForeground

3. 获取焦点（可操作）

onResume() / applicationDidBecomeActive

4. 失去焦点（还在视野内，不可操作）

onPause() / applicationWillResignActive

5. 离开视野

onStop() / applicationDidEnterBackground

6.杀死程序

onDestroy() / applicationWillTerminate


###2.在APPDelegate类实现的文件以及对应操作
```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOption{// Override point for customization after application launch.
    NSLog(@"didFinishLaunchingWithOptions");
    return YES;
}

/* 当应用程序从活动状态(active)变到非活动状态(inactive时被触发调用， 这可能发生在一些临时中断下(例如：来电话、来短信)又或者程序退出时，他会先过渡到后台然后terminate 使用这方法去暂停正在进行的任务，禁用计时器，节流OpenGL ES 帧率。在游戏中应该在这个方法里面暂停游戏。 */
- (void)applicationWillResignActive:(UIApplication *)application { 
    // Sent when the application is about to move from active to inactive state. This can occur for certain types of temporary interruptions (such as an incoming phone call or SMS message) or when the user quits the application and it begins the transition to the background state.    // Use this method to pause ongoing tasks, disable timers, and throttle down OpenGL ES frame rates. Games should use this method to pause the game.
    NSLog(@"WillResignActive");
}

 /* 使用这种方法来释放共享资源,保存用户数据,无效计时器,存储足够多的应用程序状态信息来恢复您的应用程序的当前状态,以防它终止丢失数据。 如果你的程序支持后台运行，那么当用户退出时不会调用applicationWillTerminate。 */
- (void)applicationDidEnterBackground:(UIApplication *)application {
    // Use this method to release shared resources, save user data, invalidate timers, and store enough application state information to restore your application to its current state in case it is terminated later.      // If your application supports background execution, this method is called instead of applicationWillTerminate: when the user quits.
        NSLog(@"DidEnterBackground");
    }

 /* 先从后台切换到非活动状态，然后进入活动状态。 */
- (void)applicationWillEnterForeground:(UIApplication *)application {
    // Called as part of the transition from the background to the inactive state; here you can undo many of the changes made on entering the background.
    NSLog(@"WillEnterForeground");
}

/* 重启所有的任务，不管是从非活动状态还是刚启动程序，还是后台状态。 */
- (void)applicationDidBecomeActive:(UIApplication *)application { 
    // Restart any tasks that were paused (or not yet started) while the application was inactive. If the application was previously in the background, optionally refresh the user interface.
        NSLog(@"DidBecomeActive");
    }

/* 终止，game over */
- (void)applicationWillTerminate:(UIApplication *)application { 
    // Called when the application is about to terminate. Save data if appropriate. See also applicationDidEnterBackground:.
    NSLog(@"WillTerminate");
}
```
刚启动时出现了两个：
![启动](https://upload-images.jianshu.io/upload_images/8407639-19374fc4e621cde9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

按下Home键又出现了两个：
![后台](https://upload-images.jianshu.io/upload_images/8407639-991f43415042c767.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点回去又出现了两个：
![回到前台](https://upload-images.jianshu.io/upload_images/8407639-2f09f12d0ac2bf38.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

由此，可以得到以下生命周期图：

![活动和非活动](http://upload-images.jianshu.io/upload_images/8407639-fe0102c350d8e547?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![生命周期图](http://upload-images.jianshu.io/upload_images/8407639-9d84091c2b7affdd?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对于每个阶段的分析：

- `application: didFinishLaunchingWithOptions`：程序**首次**已经完成启动时执行，一般在这个函数里面创建Window对象，将程序内容通过Window呈现给用户。

- `applicationWillResignActive`：程序将要失去`Active`状态时调用，比如有电话进来或者按下Home键，之后程序进入后台状态，对应`applicationWillEnterForeground`方法。该函数主要执行的操作有：
  a. 暂停正在执行的任务
  b. 禁止计时器
  c. 减少`OpenGL ES`帧率
  d. 若为游戏则暂停游戏

- `applicationDidEnterBackground`（已经进入后台）：对应的是`applicationDidBecomeActive`（已经变成前台）。该方法用来：
  a. 释放共享资源
  b. 保存用户数据（写到硬盘）
  c. 作废计时器
  d. 保存足够的程序状态以便下次修复

- `applicationWillEnterForeground`（即将进入前台）：用来撤销`applicationWillResignActive`（即将进入后台）做出的改变。

- `applicationDidBecomeActive`（已经进入前台）：程序已经变为`Active`时调用，对应`applicationDidEnterBackground`。若程序之前在后台，则在此方法内刷新用户界面。

- `applicationWillTerminate`：程序即将退出时调用，记得保存数据，如`applicationDidEnterBackground`一样。

###二.UIViewController生命周期
![](https://upload-images.jianshu.io/upload_images/8407639-8d6f2b7eace912e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

之前在[视频器播放项目](https://www.jianshu.com/p/2db67cf0c07b)中遇到过关于`ViewController`的问题反馈，现在再把它拿出来：

首先看看单个viewController的生命周期：
1. `loadView`：加载`view` 会多次调用并且会使`viewWillLayoutSubviews`、`viewDidLayoutSubviews`不再执行
2. **`viewDidLoad`**：`view`加载完毕
3. `viewWillAppear`：控制器的`view`将要显示
4. **`viewWillLayoutSubviews`**：控制器的`view`将要布局子控件
5. **`viewDidLayoutSubviews`**：控制器的`view`布局子控件完成
  这期间系统可能会多次调用`viewWillLayoutSubviews` 、    `viewDidLayoutSubviews` 两个方法
6. `viewDidAppear`:控制器的`view`完全显示
7. `viewWillDisappear`：控制器的`view`即将消失的时候
8. `viewDidDisappear`：控制器的`view`完全消失的时候

整个控制器生命周期：`loadView`-> **`viewDidLoad`** -> `viewWillAppear` -> **`viewWillLayoutSubviews`** -> **`viewDidLayoutSubviews`** -> `viewDidAppear` -> `viewWillDisappear` -> `viewDidDisappear`

- **`viewDidLoad`**：当程序第一次加载`view`时调用，**以后都不会用到**，而**`viewDidAppear`**是每当切换到`view`时就调用。

- **`viewWillAppear `**：系统在载入所有的数据之后，将会在屏幕上显示视图，这时候会先调用这个方法，通常我们会在这个方法对即将显示的视图做进一步的设置，如：设置设备不同**方向**的时候该如何显示；设置状态栏方向、设置视图显示样式等等。

- **`viewWillLayoutSubviews`** ：在 `viewWillAppear` 之后， `viewDidAppear` 之前执行，`view`即将布局其`subViews`，比如`view`的`bounds`发生了改变（状态栏从不显示到显示、视图方向发生变化），在**调整之前**要做的工作可以放在这个方法实现。

注意：这个方法会被调用多次，如果在此创建视图，**可能会创建多个**，而且这个方法中执行耗时操作依然会造成跳转卡顿的问题。

- **`viewDidLayoutSubviews `**：放置**调整之后**的工作

- **`didReceiveMemoryWarning`**：在内存足够的情况下，APP的视图通常会一直保存在内存中，但如果内存不够，一些没有正在显示的`View Controller`就会收到内存不足的警告，然后就会释放自己拥有的视图，以达到释放内存的目的。但是系统只会释放内存，并不会释放对象的所有权，所以通常将不需要显示在内存中保留的对象释放它的所有权，将其指针置为`nil`。

下面写个程序来试试：
```
//视图控制器中的视图加载完成，viewController自带的view加载完成
- (void)viewDidLoad {
    NSLog(@"One--%s", __FUNCTION__);
    [super viewDidLoad];
    
    UIButton *button = [[UIButton alloc] initWithFrame: CGRectMake(self.view.frame.size.width / 2 - 100,
                                                                   self.view.frame.size.height / 2 - 30,
                                                                   100,
                                                                   30)];
    [self.view addSubview: button];
    [button setTitle: @"Click" forState: UIControlStateNormal];
    [button setBackgroundColor: [UIColor blueColor]];
    [button addTarget: self action: @selector(buttonClicked:) forControlEvents: UIControlEventTouchUpInside];
    
    
    // Do any additional setup after loading the view.
}

- (void) buttonClicked: (UIButton *) sender {
    AnotherViewController *anotherViewController = [[AnotherViewController alloc] init];
    [self.navigationController pushViewController: anotherViewController animated: YES];
    
}
//出现内存警告  //模拟内存警告:点击模拟器->hardware-> Simulate Memory Warning
- (void)didReceiveMemoryWarning {
    NSLog(@"One--%s", __FUNCTION__);
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}

//视图将要出现
- (void)viewWillAppear:(BOOL)animated {
    NSLog(@"One--%s", __FUNCTION__);
    [super viewWillAppear:animated];
}

//将要布局子视图
- (void) viewWillLayoutSubviews {
    NSLog(@"One--%s", __FUNCTION__);
    [super viewWillLayoutSubviews];
}

//布局子视图完毕
- (void)viewDidLayoutSubviews {
    NSLog(@"One--%s", __FUNCTION__);
    [super viewDidLayoutSubviews];
}

//视图已经出现
- (void)viewDidAppear:(BOOL)animated {
    NSLog(@"One--%s", __FUNCTION__);
    [super viewDidAppear:animated];
}

//视图将要消失 //双击Home键,向上推出程序执行该函数
- (void)viewWillDisappear:(BOOL)animated {
    NSLog(@"One--%s", __FUNCTION__);
    [super viewWillDisappear:animated];
}

//视图已经消失
- (void)viewDidDisappear:(BOOL)animated {
    NSLog(@"One--%s", __FUNCTION__);
    [super viewDidDisappear:animated];
}
```

下面来演示一下各种情况各个步骤的执行顺序：
#####1.启动 
 `viewDidLoad` -> `viewWillAppear` -> `viewWillLayoutSubviews` -> `viewDidLayoutSubviews` -> `viewDidAppear`
![](https://upload-images.jianshu.io/upload_images/8407639-a1609e31c67a1b30.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####2.模拟Memory Warning
点击模拟器->hardware-> Simulate Memory 

![](https://upload-images.jianshu.io/upload_images/8407639-77d65abaa69fa2a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####3.跳转到Controller2
可以看到，两个`View Controller`的生命周期是很符合我们设想的逻辑的，2加载完毕后告知1准备消失，2准备出现，然后2可以进行子布局的布置，布置完毕后，1先消失，随后2出现。

 **`2--viewDidLoad`** -> `1--viewWillDisappear` -> **`2--viewWillAppear`** ->  **`2--viewWillLayoutSubviews`** -> **`2--viewDidLayoutSubviews`** -> `1--viewDidDisappear` -> **`2--viewDidAppear`**
![](https://upload-images.jianshu.io/upload_images/8407639-8a5e7c1bae5b8c40.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####4.Controller2出栈
出栈的过程其实就是免去了加载布局和布置子布局，因此他的逻辑顺序同样是消失的先执行，出现的后执行，如下所示：

**`2--viewWillDisappear`** -> `1--viewWillAppear` ->  **`2--viewDidDisappear`** -> `1--viewDidAppear`
![](https://upload-images.jianshu.io/upload_images/8407639-139c54b1813797d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
