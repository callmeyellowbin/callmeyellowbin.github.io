---
layout:     post
title:      Effective OC读书笔记三：接口与API设计
#subtitle:   学习对象初始化
date:      2018-10-16
author:     Hoben Wong
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - iOS
    - Objective-C
    - Effective OC读书笔记
---
#### 15.前缀避免命名空间冲突
- 选择与公司、应用程序等有关联作为类名的前缀，并在所有代码中使用该前缀
- 若自己所开发的程序库中用到了第三方库，则应为其中的名称加上前缀，比如XYZ引入了ABC的第三方库，则命名为XYZABCViewController。

#### 16. 提供“全能初始化方法”
为避免用户的不正确输入导致的系统崩溃（比如正方形的长宽不一致），设计者提供的接口应该考虑到用户所有的输入情况，避免产生崩溃。

提供好一个初始化方法后，如果不希望其他方法导致崩溃，则需要复写该方法：
```
//初始化方法
- (id) initWithWidth:(float)width andHeight:(float)height
{
    if ((self = [super init])) {
        _width = width;
        _height = height;
    }
    return self;
}

- (id) init {
    @throw [NSException exceptionWithName: NSInternalInconsistencyException
                        reason:@"Must use initWithWidth:andHeight:."
                        userInfo:nil];
}
```
如果有一个正方形继承了该矩形，则要求长宽一致，且禁止调用超类的方法：
```
- (id)initWithDimension:(float)dimension {
    return [super initWithWidth: dimension andHeight: dimension];
}

- (id) initWithWidth:(float)width andHeight:(float)height
{
    @throw [NSException exceptionWithName: NSInternalInconsistencyException
                        reason:@"Must use initWithDimension:."
                        userInfo:nil];
}
```

#### 17.实现description方法
如果在自定义的类使用NSLog的话，会导致对象信息缺失：
```
NSArray *array = @[@"A string", @(123)];
NSLog(@"array = %@", array);
/*array = (
    "A string",
    123
)*/

NSLog(@"object = %@", object);
//object = <EOCPerson: 0x7fd9a1600600>
```
这时候则需要自己在类里覆写description方法：
```
- (NSString *) description {
    return [NSString stringWithFormat: @"<%@: %p, \"%@ %@\">", 
            [self class], self, _firstName, _lastName];
}
// person = <EOCPerson: 0x7fd9a1600600, "Bob Smith">
```
LLDB的“po”命令可以完成对象打印（print-object）工作。

#### 18.尽量使用不可变对象
在一般情况下，我们要建模的数据未必需要改变，比如说，一些来自后台的数据，在客户端即使修改了也不会推送给服务器，那么，这些数据应当设定为不可变的，使用readonly关键词。

在定义类的公共API的时候，还需要注意：对象里表示各种collection的那些属性究竟是设成可变还是不可变的。

比如，我们用某个类来表示个人信息，该类里还存放了一些引用，指向此人的诸位朋友。那么，这个人的全部朋友就会放在一个列表里面，并将其做成属性。这种情况下，通常提供一个readonly属性供外界使用，该属性将返回不可变的set，而此set是内部那个可变set的一份拷贝。
```
//EOCPerson.h
#import <Foundation/Foundation.h>

@interface EOCPerson : NSObject

@property (nonatomic, copy, readonly) NSString   *firstName;
@property (nonatomic, copy, readonly) NSString   *lastName;
@property (nonatomic, strong, readonly) NSSet    *friends;

- (id)initWithFirstName:(NSString *)firstName
               lastName:(NSString *)lastName;

- (void)addFriends:(NSSet *)person;
- (void)removeFriends:(NSSet *)person;

@end
```
```
//EOCPerson.m
#import "EOCPerson.h"
@interface EOCPerson()

@property (nonatomic, copy, readwrite) NSString *firstName;
@property (nonatomic, copy, readwrite) NSString *lastName;

@end

@implementation EOCPerson {
    NSMutableSet *_internalFriends;
}

-(id)initWithFirstName:(NSString *)firstName lastName:(NSString *)lastName {
    if (self = [super init]) {
        self.firstName = firstName;
        self.lastName = lastName;
        _internalFriends = [[NSMutableSet alloc] init];
    }
    return self;
}

-(NSSet *)friends {
    return [_internalFriends copy];
}

- (void)addFriends:(NSSet *)person {
    [_internalFriends addObject:person];
}

- (void)removeFriends:(NSSet *)person {
    [_internalFriends removeObject:person];
}

@end
```
当我们需要提供API时，要提供一个完整的接口，而不是让用户直接操作我们的底层来使用，因此，对外的接口不要用NSMutableSet来实现friends属性，这样会导致在EOCPerson对象不知情的情况下，直接从底层修改了set，令对象内的各数据之间互不一致。

要点如下：
- 尽量创建不可变对象。
- 若某属性仅可于对象内部修改，则在扩展中将其由readonly属性扩展为readwrite属性
- 不要把可变的collection作为属性公开，应提供相关方法，以此修改对象中的可变collection

#### 19.理解OC错误模型
ARC在默认情况下不是异常安全的，即，抛出异常以后，本应该在作用域末尾释放的对象却不会自动释放了。因此，异常只应该用于极其严重的错误，比如编写了某个抽象基类，它的正确用法是先从中继承一个子类，然后使用这个子类。如果有人直接使用了这个抽象基类，则可以考虑抛出异常。

如果是非致命错误，则可以令方法返回nil/0,以表明其中有错误发生。

```
- (id)initWithValue:(id)value {
    if ((self = [super init])) {
        if (/* Value means instance can't be created */) {
            self = nil;
        } else {
            // Initialize instance
        }
    }
    return self;
}
```

也可以采用灵活的NSError，经由此对象，我们可以把导致错误的原因回报给调用者，它封装了三条信息：
- *Error domain*（错误范围，类型为字符串）
即产生错误的根源，通常用一个特有的全局变量来定义。比如，处理URL的子系统在从URL中解析或取得数据时如果出错了，就会使用`NSURLErrorDomain`来表示错误范围。

- *Error code*（错误码，类型为整数）
独有的错误代码，用以指明在某个范围内发生了何种错误。某个特定范围内可能会发生一系列相关错误，这些错误情况通常用enum来定义。例如，当HTTP请求出错时，可能会把HTTP状态码设为错误码。

- *User info*（用户信息，类型为字典）
有关此错误的额外信息，其中或许包含一段“本地化的描述”，或者还含有导致该错误发生的另外一个错误。

在设计API的时候，`NSError`的第一种常见用法就是通过委托协议来传递此错误，有错误发生时，当前对象会把错误信息经由协议中的某个方法传给其委托对象，如`NSURLConnection`在其委托协议`NSURLConnectionDelegate`之中就定义了如下方法：

```
- (void)connection:(NSURLConnection *) connection
        didFailWithError:(NSError *) error
```

当`NSURLConnection`出错之后，就会调用此方法以处理相关错误。调用者获得error，从而获取里面的信息，调用者可以自己决定是否回报此错误，比抛出异常要好。

```
- (void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error{

    if (self.completionBlock) {

       NSError* error = [NSErrorerrorWithDomain:error.domain

                                            code:error.code

                                        userInfo:error.userInfo];

       self.completionBlock(error);

    }

    _isFinish = YES;

    [self.osclose];

}
```

NSError的另一种常见用法是：经由方法的“输出参数”返回给调用者。如：

```
- (BOOL)doSomething:(NSError **) error
```

传递给方法的参数是个指针，而该指针本身又指向另一个指针，那个指针指向NSError对象。或者也可以把它当场一个直接指向NSError对象的指针。这样一来，此方法不仅能有普通的返回值，还能经由“输出参数”把NSError对象回传给调用者。用法如下：

```
NSError *error = nil;
BOOL ret = [object doSomething:&error];
if (error) {
    //There was an error
}
```

当然，也可以不关注具体的错误信息：

```
BOOL ret = [object doSomething:nil];
if (ret) {
    //There was an error
}
```

在我们定义这个方法的时候，要这样写：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    NSError *error = nil;
    BOOL ret = [self doSomething:&error];
    if (ret) {
        //There was an error
    }
}

- (void)doSomething:(NSError **)error {
    if (/* There was an error */) {
        if (error) {
            error = [NSError errorWithDomain:domain
                                        code:code
                                    userInfo:userInfo];
        }
        return NO;
    } else {
        return YES;
    }
}
```

在这里，\&\*error代表着error指针所在的地址，我们要确保这个地址不为空，因为对空指针解引用会导致段错误，使应用程序崩溃，比如如果传入了一个nil，如果不加以判断就会产生段错误。但是，如果error指针的地址不为空（即\*error = nil），这时候就可以更改\*error指向的对象，从而指向真正的error。

错误码可以定义为枚举类型：

```
//.h
extern NSString *const EOCErrorDomain;
typedef NS_ENUM(NSUInteger, EOCError) {
    EOCErrorUnknown                 = -1,
    EOCErrorInternalInconsistency   = 100,
    EOCErrorGeneralFault            = 105,
    EOCErrorBadInput                = 500,
};

//.m
NSString *const EOCErrorDomain = @"EOCErrorDomain";
```

#### 20.理解NSCopying协议
若想某个类支持拷贝功能，只需声明该类遵从`NSCopying 协议 `，并实现其中的那个方法即可。如个人信息的拷贝：

```
- (id)copyWithZone:(NSZone *)zone {
    EOCPerson *copy = [[[self class] allocWithZone: zone]
                       initWithFirstName:_firstName
                       lastName:_lastName];
    copy->_internalFriends = [_internalFriends mutableCopy];
    return copy;
}
```

- *为什么需要这样来实现copy方法？*
如果令两个对象直接共享一个可变的set方法的话，给原来的对象添加一个新朋友之后，另一个对象也会随之添加。如果set是不可变的，则无需复制，内容不变的情况下无需考虑。

当然也有`NSMutableCopying 协议 `，如果你的类分为可变版本和不可变版本，就应该实现这个协议：

```
- (id)mutableCopyWithZone:(NSZone*)zone
```

对于`copy`和`mutableCopy`，有以下关系：

```
[NSMutableArray copy] => NSArray
[NSArray copy] => NSMutableArray
```

应当要注意判断需要拷贝的类型，再调用相应的方法。

只有对不可变对象进行copy操作是浅复制（即指针复制），其他情况下都是深复制（内容复制），即`copy可变对象`、`mutableCopy不可变对象`和`mutableCopy可变对象`。

浅复制和深复制的区别在于复制的是指针（处于栈区）还是内容（处于堆区），它们的区别可以参考[C++/Java/OC的内存管理](https://www.jianshu.com/p/1e52428695cf)，浅复制的指针指向的内容一旦发生改变，则复制得到的东西也会发生变化，这点需要注意。

值得注意的是，即使用到了copy，也要分为`集合类对象`（NSArray、NSDictionary）和`非集合类对象`（NSString）。

- copy非集合类，则规则和之前提到的规则一样。

- copy集合类，则对于集合类的深复制仅限本身，集合类里面的元素是浅复制（如@[@"1", @"2"]中，整个数组可以被深复制，但是数组里面的内容（“1”，“2”）仅会复制指针）。
