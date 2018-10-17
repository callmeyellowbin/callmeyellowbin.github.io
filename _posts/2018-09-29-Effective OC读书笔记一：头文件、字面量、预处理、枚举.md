---
layout:     post
title:      Effective OC读书笔记一：头文件、字面量、预处理、枚举
#subtitle:   学习对象初始化
date:      2018-09-29
author:     Hoben Wong
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - iOS
    - Objective-C
    - Effective OC读书笔记
---
#### 1.在类的头文件中尽量少引入其他头文件
- 向前声明：@class Subclass，无需知道Subclass的全部细节，只需要知道有个类名叫Subclass即可，只有在确有需要时才会引入，减少了类的使用者所需引入的头文件数量。
- 引入头文件：#import < Subclass.h>，必须知道其所有接口的细节，增加了编译时间，且容易造成循环引用，导致两个类中有一个无法被正确编译。

要点：
- 除非确有必要，否则不要引入头文件。一般来说，应在某个类的头文件中使用`向前声明`来提及别的类，并在实现文件中引入那些类的头文件。可以降低类之间的耦合。
- 有时无法使用向前声明，比如要声明某个类遵循一项协议<Protocol>。这种情况下，尽量把“该类遵循某协议”这条声明移至Extension中，如果不行，则将协议单独放在一个头文件中，然后将其引入。

#### 2.多用字面量语法，少用与之等价的方法
- 字面量语法：@"String"、@1、@[@"cat",@"dog"]、animals[1]

要点：
- 使用字面量语法来创建字符串、数值、数组、字典
- 通过取下标操作来访问数组下标
- 用字面量语法创建数组或者字典时，值中不能有nil，否则会抛出异常

#### 3.多用类型常量，少用#define预处理指令
- (写在.m) \#define kDuration 0.3：会把源代码中碰到的kDuration字符串`一律`替换成0.3，所有引入了这个头文件的代码，其kDuration都会被替换。
- (写在.m) static const NSTimeInterval kDuration = 0.3：定义了一个NSTimeInterval的常亮，更清楚地描述了常量的含义。
- (写在.h) extern const  NSTimeInterval kDuration、(写在.m) const NSTimeInterval kDuration = 0.3：告诉编译器，在全局符号表中有一个名叫kDuration的符号，编译器无须查看其定义，即允许代码使用此常量。此类常量必须要定义，而且只能定义一次。一旦在.m文件定义好，即可随处使用。

注意：变量一定要同时用static和const来声明，如果试图修改const修饰符所声明的变量，那么编译器会报错。

而static修饰符意味着该变量仅在定义次变量的编译单元中可见。编译器每收到一个编译单元，就会输出一份目标文件（在OC语境下，编译单元通常指的是每个类的实现文件，即.m）。

假如此变量不加static，编译器就会为他创建一个外部符号，如果另一个编译单元也声明了同名变量，则会报错。

要点：
- 不要用预处理指令定义常量。这样定义出来的常量不含类型信息，编译器只会在编译前据此执行查找和替换操作。即使有人重新定义了常量值，编译器也不会产生警告信息，导致应用程序中的常量值不一致。
- 在实现文件（.m）中使用static const来定义`只在编译单元内可见的常量`，由于此类常量不在全局符号表中，所以无须为其名称加前缀。
- 在头文件中使用extern来声明`全局变量`，并在相关实现文件中定义其值，这种常量要出现在全局符号表中，所以其名称应加以区隔，通常用与之相关的类名作为前缀。

#### 4.用枚举表示状态、选项、状态码
- enum：编译器会为枚举分配一个独有的编号，从0开始，每个枚举递增1。如果需要定义选项，则最好自己赋值，凡是需要以按位或操作来组合的枚举都应使用NS_OPTIONS定义。每个选项都可以选用或者禁用，根据|来操作，示例如下：
```
typedef NS_OPTIONS(NSUInteger, UIViewAutoresizing) {
    UIViewAutoresizingNone                 = 0,
    UIViewAutoresizingFlexibleLeftMargin   = 1 << 0,
    UIViewAutoresizingFlexibleWidth        = 1 << 1,
    UIViewAutoresizingFlexibleRightMargin  = 1 << 2,
    UIViewAutoresizingFlexibleTopMargin    = 1 << 3,
    UIViewAutoresizingFlexibleHeight       = 1 << 4,
    UIViewAutoresizingFlexibleBottomMargin = 1 << 5
};
```
若是枚举不需要相互结合，则应使用NS_ENUM来定义。
```
typedef NS_ENUM(NSUInteger, EOCConnectionState) {
    EOCConnectionStateDisconnected,
    EOCConnectionStateConnecting,
    EOCConnectionStateConnected
};
```
使用Switch来定义枚举（使用状态机时最好不写default分支，并确保Switch语句能正确处理NS_ENUM里面的所有样式）：
```
switch (_currentState) {
    EOCConnectionStateDisconnected:
      //...
      break;
    EOCConnectionStateConnecting:
      //...
      break;
    EOCConnectionStateConnected:
      //...
      break;
}
```
要点：
- 应该用枚举来表示状态机的状态、传递给方法的选项以及状态码等值，给这些值起个易懂的名字。
- 如果把传递给某个方法的选项表示为枚举类型，而**多个选项又可以同时使用**，则将各选项值定义为2的幂，以便通过按位|操作将其结合起来。
- 使用NS_ENUM与NS_OPTIONS宏来定义枚举类型，并指明其底层数据类型。这样做可以确保枚举是用开发者所选的底层数据类型实现出来的，而不会采用编译器所选的类型。
- 在处理枚举类型的Switch语句不要实现default分支。这样的话，加入新枚举以后，编译器就会提示开发者：Switch语句并未处理所有枚举。
