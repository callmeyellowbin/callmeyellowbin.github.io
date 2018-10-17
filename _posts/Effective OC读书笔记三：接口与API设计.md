---
layout:     post
title:      Effective OC读书笔记三：接口与API设计
#subtitle:   学习对象初始化
date:      2018-06-08
author:     Hoben Wong
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - iOS
    - Objective-C
    - Effective OC读书笔记
---
####15.前缀避免命名空间冲突
- 选择与公司、应用程序等有关联作为类名的前缀，并在所有代码中使用该前缀
- 若自己所开发的程序库中用到了第三方库，则应为其中的名称加上前缀，比如XYZ引入了ABC的第三方库，则命名为XYZABCViewController。

####16. 提供“全能初始化方法”
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

####17.实现description方法
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

####18.尽量使用不可变对象
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
