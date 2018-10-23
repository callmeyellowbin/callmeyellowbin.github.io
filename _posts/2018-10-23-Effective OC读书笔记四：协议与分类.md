---
layout:     post
title:      CSS学习笔记
#subtitle:   学习对象初始化
date:      2018-10-23
author:     Hoben Wong
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - iOS
    - Objective-C
    - Effective OC读书笔记
---

#### 23. 对象间通信：委托、数据源
委托模式可以将数据与业务逻辑解耦。比如：用户界面里有个显示一系列数据所用的视图，那么，此视图只应包含显示数据所需的逻辑代码，而不应决定要`显示何种数据`以及`数据之间如何交互`等问题。视图对象的属性中，可以包含`负责数据`与`事件处理`的对象，分别为`数据源`和`委托`。

例：假设要编写一个从网上获取数据的类，此类也许要从远程服务器的某个资源里获取数据，那个远程服务器过很长时间才能应答，所以不能再获取数据的过程中阻塞应用程序。

做法：获取网络数据的类含有一个`委托对象`，在获取完数据之后，它会回调这个委托对象。如下图，`EOCDataModel`就是`EOCNetworkFetcher`的委托对象，`EOCDataModel`请求`EOCNetworkFetcher`用异步方式执行一项任务，而`EOCNetworkFetcher`执行完这项任务之后，就会通知其委托对象`EOCDataModel`。

![](https://upload-images.jianshu.io/upload_images/8407639-90059015e17631ab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
@protocol EOCNetworkFetcherDelegate

- (void)networkFetcher:(EOCNetworkFetcher *)fetcher
        didReceiveData:(NSData *)data;

@end
```

```
@interface EOCNetworkFetcher:NSObject

@property (nonatomic, weak) id <EOCNetworkFetcherDelegate> delegate;

@end

@implementation EOCNetworkFetcher
...

NSData *data = /* data from network */;
if ([self.delegate respondsToSelector:@selector(networkFetcher:didReceiveData:)]) {
    [self.delegate networkFetcher: self didReceiveData: data];
}

...
@end
```

```
@implementation EOCDataModel () <EOCNetworkFetcherDelegate>
@end

@implementation EOCDataModel
...

_fetcherA.delegate = self;
_fetcherB.delegate = self;

- (void)networkFetcher:(EOCNetworkFetcher *)fetcher
        didReceiveData:(NSData *)data {
    if (fetcher == _fetcherA) {
        /* Handle data*/
    } else if (fetcher == _fetcherB) {
        /* Handle data*/
    }
}

...
```

- 委托模式：对象把应对某个行为的责任委托给另外一个类。
- 数据源模式：用协议定义一套接口，令某类经由该接口获取其所需的数据。

![](https://upload-images.jianshu.io/upload_images/8407639-b08fb656e4a6b760.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在`ListView`中，会通过数据源协议来获取`要在列表中显示的数据`。还有一个受委托者，用于处理`用户和列表的交互操作`。将数据源协议和委托协议分离，能使接口更加清晰，这两部分的逻辑代码也分开了。一般情况下由同一个对象来扮演`数据源`和`受委托者`。

如果需要频繁地调用委托模式，会重复查询@selector，这时候需要对查询结果进行缓存。

```
@interface EOCNetworkFetcher () {
    struct {
        unsigned int didReceiveData:1;
        unsigned int didFailWithError:1;
    } _delegateFlags;
}
@end

- (void)setDelegate:(id<EOCNetworkFetcher>)delegate {
    _delegate = delegate;

    _delegateFlags.didReceiveData = 
        [delegate respondsToSelector:@selector(networkFetcher:didReceiveData:)];

    _delegateFlags.didFailWithError = 
        [delegate respondsToSelector:@selector(networkFetcher:didFailWithError:)];
}

if (_delegateFlags.didReceiveData) {
    [_delegate networkFetcher: self didReceiveData: data];
}
```

#### 24. 将类的实现代码分散到便于管理的数个分类之中
- 使用分类机制把类的实现代码划分成易于管理的小块。
- 将应该视为“私有”的方法归入名叫Private的分类中，以隐藏实现细节。

#### 25. 总是为第三方类的分类名称加前缀
- 向第三方类添加分类时，总应给其名称和方法名加上专用的前缀。

#### 26. 勿在分类中声明属性
在分类中无法合成属性相关的实例变量，所以开发中需要在分类中为该属性实现存取方法。

```
#import <objc/runtime.h>

static const char *kFriendsPropertyKey = "kFriendsPropertyKey";

@implementation EOCPerson (Friendship)

- (NSArray *) friends {
    return objc_getAssociatedObject(self, kFriendsPropertyKey);
}

- (void) setFriends:(NSArray *) friends {
    objc_setAssociatedObject(self, 
                             kFriendsPropertyKey,
                             friends,
                             OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

@end
```

但是，这样做要把相似的代码写很多遍，而且在内存管理问题上容易出错，因为我们在为属性实现存取方法时，经常会忘记遵从其内存管理语义。比如，可以通过属性的`attribute`修改某个属性的内存管理语义。而此时还要记得，在设置方法中也得修改设置关联对象时所用的内存管理语义才行。

正确做法应该是把所有属性都定义在主接口中。类封装的全部数据都应定义在主接口中，这里是唯一能够定义实例变量的地方。

```
@interface NSCalendar (EOC_Additions)

//YES
- (NSArray *) eoc_allMonths;

//NO
//@property (nonatomic, strong, readonly) NSArray *eoc_allMonths;

@end
```

#### 27. 使用扩展隐藏实现细节
扩展可以避免暴露接口，把我们需要的实例变量放在.m或者.mm文件中，在此不再赘述。

由于兼容性原因，Objective-C++是OC和C++的混合体，其代码可以用这两种语言编写，文件扩展名要改为.mm。但是如果写在.h文件，就会导致包含这个头文件的所有文件都要改成.mm，这是极其麻烦的事情，因此，使用扩展显得非常必要。

```
// EOCClass.mm

#import "EOCClass.h"

#include "SomeCppClass.h"

@interface EOCClass() {
    SomeCppClass _cppClass;
}
@end

@implementation EOCClass
@end
```

在实现协议时，如果实现的是私有的内部协议，也要使用扩展来声明：

```
@interface EOCPerson() <EOCSecretDelegate>
@end
```

#### 28. 通过协议提供匿名对象
匿名对象：任何类的对象都能承当这一属性，即便该类没有继承NSObject。

```
@property (nonatomic, weak) id <EOCDelegate> delegate;
```

除了委托，在数据库查询中，匿名对象也非常常见，因为处理连接所用的那个类，如果不想让外人知道其名字，因为不同的数据库可能要用不同的类来处理。如果没办法令其都继承自同一积累，那么就得返回id类型的东西了。不过我们可以把所有数据库连接都具备的那些方法放到协议中，令返回的对象遵循此协议。

```
@protocol EOCDatabaseConnection
- (void) connect;
- (void) disconnect;
- (BOOL) isConnected;
- (NSArray *) performQuery:(NSString *) query;
```

然后就可以用数据库处理器单例来提供数据库连接了。

```
@protocol EOCDatabaseConnection;

@interface EOCDatabaseManager: NSObject
+ (id) sharedInstance;
- (id<EOCDatabaseConnection>)connectionWithIdentifier:(NSString *)identifier;

@end
```

这样的话，处理数据库连接所用的类的名称就不会泄漏了，有可能来自不同框架的那些类现在均可以经由同一个方法来返回了。使用此API的人仅仅要求所返回的对象能够用来连接、断开并查询数据库即可。

在上面的例子中，处理数据库连接所用的后端代码可能使用了各种第三方库来连接不同类型的数据库，由于这些类都在多个第三方库中，所以也许没办法令所有的连接类都继承于同一基类。因此，可以创建匿名对象把这些第三方类简单包裹一下，使匿名对象成为其子类，并遵从`EOCDatabaseConnection`协议，用`connectionWithIdentifier:`方法返回这些类对象。在开发后续版本时，无须改变公共API，即可切换后端的实现类。

有时候如果说对象类型不重要，重要的是对象有没有实现某些方法，也可以实现匿名类型来表达这一概念。可以写成遵从某协议的匿名类型，以表示类型在此处不重要。

例：CoreData框架，查询CoreData数据库所得的结果由名叫`NSFetchedResultsController`的类来处理，如有需要，处理时还会用`section`属性把数据分区。此属性是个数组，但其中的对象却没有指明具体类型，只是说这些对象都遵从了`NSFetchedResultsSectionInfo`协议。

```
NSFetchedResultsController *controller;
NSUInteger section;

NSArray *sections = controller.sections;
id < NSFetchedResultsSectionInfo> sectionInfo = sections[section];
NSUInteger numberOfObjects = sectionInfo. numberOfObjects;
```

`sectionInfo`是匿名对象，在幕后，该对象可能是由处理结果的控制器所创建的内部状态对象。没必要把表示此种数据的类对外公布，因为使用控制器的人绝对不用关心查询结果中的数据分区是如何保存的，他们只需要知道可以在这些对象上查询数据就行了。
