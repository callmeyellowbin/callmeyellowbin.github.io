---
layout:     post
title:      HelloWorld！
#subtitle:   学习对象初始化
date:      2018-06-06
author:     Hoben Wong
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - iOS
    - Objective-C
---
### 一.项目文件
![导航视图](https://upload-images.jianshu.io/upload_images/8407639-f71360661decb06d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我们从上到下看看这些文件是干嘛的。
- Hello World
  这是第一个文件夹，以项目名来命名，许多工作都要在这个文件夹中完成。它包含了应用的大部分代码以及用户界面文件夹，我们可以在这个文件夹下创建任意个子文件夹，甚至可以使用其他分组代替这个默认的文件夹，从而更好地组织代码。
  其中，里面的Main.storyboard包含了项目主视图控制器用到的用户界面元素。
- Hello WorldTests
  包含项目中必需的源代码文件和资源文件。
  Info.plist：包含了应用程序的重要信息，例如名称，对运行的设备规格是否有要求，等等。
  main.m：通常不需要编辑或者修改。
- Hello WorldUITests
  用于编写一些单元测试代码，文件夹内包含了所需的初始化文件。
- Products：包含构建项目时生成的应用。Hello World.app即这个项目构建的文件，为红色即这个文件不存在。
### 二.Interface
StoryBoard里面有View Controller(视图控制器)、First Responder(第一响应者)和Exit。
- 视图控制器代表一个控制器对象，会从文件中加载控制器以及相关的视图。它的任务是管理用户在屏幕上看到的内容。一个应用程序通常有多个视图控制器，每个界面各一个。也可以编写仅有一个界面的应用程序，这样就仅有一个视图控制器。
- 第一响应者即用户当前正在交互的对象。如果用户正在向一个文本框输入数据，则该文本框就是当前的第一响应者。他会随着用户与用户界面的交互而变化，不需要编写代码来判断哪个控件（或视图）是第一响应者。
### 三.资源库
包含了4个部分：
- 文件模板库：包含一些文件模板，可以通过它们向项目中添加新文件。
- 代码片段库：包含一些代码片段，可以直接把它们拖到源代码中使用。
- 对象库：包括各种可重用对象，如文本框、标签、滑块、按钮等用来设计iOS界面的对象。
- 媒体库：包含用户的所有媒体文件，有图片、声音以及影片文件等。
- 底部搜索框：可以搜索任何想要的控件
  ![资源库的四个部分](https://upload-images.jianshu.io/upload_images/8407639-a14558fc4a9d714d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 四.添加标签
将label直接拖动到View Controller中央。
![](https://upload-images.jianshu.io/upload_images/8407639-484f17b7d1e7188d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这样就完成了我们的Hello World啦！
![](https://upload-images.jianshu.io/upload_images/8407639-d032aa9bb1cafe46.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 五.交互
在ViewController.h中，我们需要定义一个Button：
```
#import <UIKit/UIKit.h>

@interface ViewController : UIViewController

@property  (weak, nonatomic) IBOutlet UIButton *myButton;

- (IBAction) doSomething;

- (IBAction) doSomething: (id)sender;

@end
```
其中，sender的作用是获得触发该方法的对象。
![打开Assistant Editor](https://upload-images.jianshu.io/upload_images/8407639-c8247006e28c4abd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

首先，在storyboard里面新建left和right的button控件，右键至ViewController.h当中，即可新建一个按键控件类。然后再右键至doSomething方法中，表明这个按键和该方法建立了连接。
![](https://upload-images.jianshu.io/upload_images/8407639-205adf133f7f3dc0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
在ViewController.m里声明实现方法。
```
- (IBAction) doSomething:(id)sender
{
    //获得控件的名称
    NSString *title = [sender titleForState: UIControlStateNormal];
    //label要显示的内容
    NSString *plainText = [NSString stringWithFormat: @"%@ button pressed.", title];
    _stausLabel.text = plainText;
}
```
#### 2.限制布局
我们用右键，从View拖到对应的子控件去控制其约束。
![](https://upload-images.jianshu.io/upload_images/8407639-95e5e544d33fe9dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
选中对应的子控件后，点击相应的方法，Horizontally即水平居中，Vertically即垂直居中。
![](https://upload-images.jianshu.io/upload_images/8407639-fa1e18ba916d621d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
随后调整该控件到顶部的距离。
![](https://upload-images.jianshu.io/upload_images/8407639-bace1ca4564716a6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
同理，两个按钮也是这样搞。可以看到布局就这样搞定了！
![结果展示](https://upload-images.jianshu.io/upload_images/8407639-aa126dae55c70158.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
