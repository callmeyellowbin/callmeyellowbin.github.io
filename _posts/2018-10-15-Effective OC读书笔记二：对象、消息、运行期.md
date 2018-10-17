---
layout:     post
title:      Effective OC读书笔记二：对象、消息、运行期
#subtitle:   学习对象初始化
date:      2018-10-15
author:     Hoben Wong
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - iOS
    - Objective-C
    - Effective OC读书笔记
---
#### 6.理解属性这一概念
> Property = ivar + getter + setter

- 在OC中，实例变量是一种存储偏移量所用的“特殊变量”，交由“类对象”保管，偏移量会在运行期查找，如果类的定义变了，那么存储的偏移量也就变了。这样，无论何时访问实例变量，总能使用正确的偏移量。甚至可以在运行期向类中新增实例变量。

- 使用@dynamic关键字，会告诉编译器：不要自动创建实现属性所用的实例变量，也不要为其创建存取方法。比如，一些属性的数据来自于后端的数据库中，则需要定义为@dynamic。

- `getter = <name>` 指定“获取方法”的方法名，比如一个属性是Bool类型，前缀要加上is，则可以定义为：
```
@property (nonatomic, getter = isOn) BOOL on;
```

- 在实现存取方法时，应该保证其具备相关属性声明的特质，如，若将某个属性声明为copy，则应该在“设置方法”中拷贝相关对象，否则会误导该属性的使用者。

```
@interface EOCPerson: NSManagedObject

@property (nonatomic, copy) NSString *firstName;
@property (nonatomic, copy) NSString *lastName;

- (id)initWithFirstName:(NSString*) firstName
              lastName:(NSString*) lastName;

- (id)initWithFirstName:(NSString*) firstName
              lastName:(NSString*) lastName 
{
    if ((self = [super init])) {
        _firstName = [firstName copy];
        _lastName = [lastName copy];
    }
    return self;
}
```

#### 7. 在对象内部尽量直接访问实例变量
- 在对象内部读取数据时，应该直接通过实例变量来读，而写入数据时，则应通过属性来写。
- 在初始化方法及dealloc方法中，总是应该直接通过实例变量来读写数据。
- 有时会使用懒加载配置某份数据，这种情况下，需要通过属性来读取数据。

#### 8. 用“类族模式”隐藏实现细节
类族可以隐藏抽象基类背后的实现细节，OC的系统框架中普遍使用此模式。

比如，设置一系列按钮的时候，鉴于它们都继承于同一个基类：UIButton，则该类的使用者无须关系创建出来的按钮具体属于哪个子类，也不用考虑按钮的绘制方式等实现细节。使用者只需要明白如何创建按钮，如何设置属性，如何添加触摸动作等。

我们可以将这些按钮重构为多个子类，在类族模式中，可以灵活应对多个类，将他们的实现细节隐藏在抽象基类后面，以保持接口简介。用户无需自己创建子类实例，只需调用基类方法来创建即可。

例：有一个处理雇员的类，每个雇员都有名字和薪水两个属性，管理者可以命令其执行日常工作。但是，各个雇员的工作内容不同，经理在带领雇员做项目的时候，无需关心每个人如何完成工作，仅需指示其开工即可。
```
//  ECOEmployee.h
typedef NS_ENUM(NSUInteger, ECOEmployeeType) {
    ECOEmployeeTypeDeveloper,
    ECOEmployeeTypeDesigner,
    ECOEmployeeTypeFinance,
};

@interface ECOEmployee : NSObject

@property (nonatomic, copy) NSString *name;
@property (nonatomic, assign) NSUInteger salary;

//创建Employee对象
+ (ECOEmployee *) employeeWithType:(ECOEmployeeType)type;

- (void)doADaysWork;

- (void)introduceMyself;

@end
```
```
//  ECOEmployee.m
#import "ECOEmployee.h"
#import "ECOEmployeeFinance.h"
#import "ECOEmployeeDesigner.h"
#import "ECOEmployeeDeveloper.h"

@implementation ECOEmployee

+ (ECOEmployee *)employeeWithType:(ECOEmployeeType)type {
    switch (type) {
        case ECOEmployeeTypeDeveloper: {
            return [[ECOEmployeeDeveloper alloc] init];
            break;
        }
        case ECOEmployeeTypeDesigner: {
            return [[ECOEmployeeDesigner alloc] init];
            break;
        }
        case ECOEmployeeTypeFinance: {
            return [[ECOEmployeeFinance alloc] init];
            break;
        }
    }
}

- (void)doADaysWork {
    //子类来实现
}

- (void)introduceMyself {
    NSLog(@"Hello, my name is %@ with salary %lu", self.name, self.salary);
}

@end
```
```
//  ECOEmployeeDeveloper.m
- (void)doADaysWork {
    self.name = @"developer";
    self.salary = 1000;
    [self introduceMyself];
    [self writeCode];
}

- (void)writeCode {
    NSLog(@"I am writing code!");
}
```
```
//  ECOEmployeeDesigner.m
- (void)doADaysWork {
    self.name = @"designer";
    self.salary = 2000;
    [self introduceMyself];
    [self design];
}

- (void)design {
    NSLog(@"I am designing!");
}
```
```
//  ECOEmployeeFinance.m
- (void)doADaysWork {
    self.name = @"finance";
    self.salary = 3000;
    [self introduceMyself];
    [self calculate];
}

- (void)calculate {
    NSLog(@"I am calculating!");
}
```

#### 9.在既有类中使用关联对象存放自定义数据
如果需要在对象中存放相关信息，我们通常会从对象所属的类中继承一个子类，然后该用这个子类对象。有时候类的实例由某种机制所创建的，而开发中无法令这种机制创建出自己所写的子类实例，这时候就要使用“关联对象”。

可以给某对象关联许多其他对象，通过“键”来区分，存储对象值时，可以指明“存储策略”，用以维护相应的“内存管理语义”。用以下方法来管理关联对象：
```
//以给定的键和策略为某对象设置关联对象值
void objc_setAssociatedObject(id object, void *key, id value, objc_AssociationPolicy policy)

//根据给定的键从某对象获取相应的关联对象值
id objc_getAssociatedObject(id object, void *key)

//移除对象的全部关联对象
void objc_removeAssociatedObjects(id object)
```
在设置关联对象值时，若想令两个键匹配到同一个值，则两者必须是完全相同的指针才行。所以，在设置关联对象值时，通常使用静态全局变量做键。

#### 10.理解objc_msgSend的作用
```
//OC
id returnValue = [someObject messageName:parameter];

//转化为C后
void objc_msgSend(id self, @selector(messageName:), parameter);
```
objc_msgSend函数会根据接收者与选择子的类型来调用适当的方法。为了完成此操作，该方法需要在接收者所属的类中搜寻其“方法列表”，如果能找到与选择子名称相符的方法，就跳到其实现代码，如果找不到，则沿着集成体系继续向上查找，等找到合适的方法之后再跳转。如果最终还是找不到相符的方法，则执行“消息转发”操作。

![](https://upload-images.jianshu.io/upload_images/8407639-d967cc2b9ac39aa4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

前面的情况只描述了部分消息的调用过程，其他边界情况则需要交由OC运行环境中的另一些函数来处理：
- objc_msgSend_stret: 如果待发送的消息要返回结构体，则可交由该函数处理。只有当CPU的寄存器能够容纳得下消息返回类型时，这个函数才能处理此消息。若是返回值无法容纳于CPU寄存器中（如返回的结构体太大了），则由另一个函数执行派发。此时，那个函数会通过分配在栈上的某个变量来处理消息所返回的结构体。

- objc_msgSend_fpret:如果消息返回的是浮点数，则可交由此函数处理。在某些架构的CPU中调用函数时，需要对“浮点数寄存器”做特殊处理。

- objc_msgSendSuper:如果要给超类发消息，例如[super message:parameter]，则交由此函数处理。

#### 11.理解消息转发机制
##### 动态方法解析

对象在收到无法解读的消息后，首先调用其所属类的下列类方法：
```
+ (BOOL)resolveInstanceMethod:(SEL)selector
```
该方法的参数就是未知的选择子，返回值为BOOL类型，表示这个类是否能新增一个实例方法用以处理该选择子，如果是类方法则为`resolveClassMethod:`

使用这种方法的前提是相关方法的实现代码已经写好，只等着运行的时候动态插在类里面即可。此方案常用来实现@dynamic属性，如访问CoreData中的NSManagedObject对象的属性就可以这么做，因为实现这些属性所需的存取方法在编译器就能确定。
```
+ (BOOL)resolveInstanceMethod:(SEL)sel {
    NSString *selString = NSStringFromSelector(sel);
    if (/* selector is from a @dynamic property */)  {
        if ([selString hasPrefix: @"set"]) {
            class_addMethod(self, sel, (IMP)autoSetter, "v@:@");
        } else {
            class_addMethod(self, sel, (IMP)autoGetter, "@@:");
        }
        return YES;
    }
    return [super resolveInstanceMethod:sel];
}
```

##### 备援接收者
当前接收者还有第二次机会能处理未知的选择子，在这一步中，运行期系统会问：能不能把这条消息转给其他接收者来处理：
```
- (id)forwardingTargetForSelector:(SEL)aSelector
```
如果能找到备援对象，则将其返回，若找不到，则返回nil。通过此方案，我们可以用组合来模拟出多重继承的某项特性：在一个对象内部，可能还有一系列其他对象，该对象可以经由此方法将能够处理某选择子的相关内部对象返回。在外界看来，好像是该对象亲自处理了这些消息一样。

##### 完整的消息转发
到了这一步，唯一能做的就是启用完整的消息转发机制了。首先创建NSInvacation对象，把与尚未处理的那条消息有关的全部细节都封在其中。包括选择子、目标、参数，触发NSInvocation对象时，消息派发系统将消息指派给目标对象：
```
- (void)forwardInvocation:(NSInvocation *)anInvocation
```
只需改变调用目标，使消息在新目标上得以调用即可，这个方法和之前那个方法等效，所以很少采用如此简单的实现方式。
![](https://upload-images.jianshu.io/upload_images/8407639-6b61bba5a62357e9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 12. 方法调配技术调试黑盒方法
![](https://upload-images.jianshu.io/upload_images/8407639-fa47985d28596da1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/8407639-e5f4fc50e6cc7356.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在一些已经封装好的方法中，要为其添加新方法（如加上调试记录），则可以采用Method swizzling。如：为NSString添加记录信息，可以将新方法添加到NSString的一个分类中。

```
@interface NSString (ECOMyAdditions)
- (NSString *)eoc_myLowercaseString;
@end

@implementation NSString (ECOMyAdditions)
- (NSString *)eoc_myLowercaseString {
    NSString *lowercase = [self eoc_myLowercaseString];
    NSLog(@"%@ => %@", self, lowercase);
    return lowercase;
}
@end
```
在这段代码中，似乎会陷入递归的死循环，但是通过Method swizzling，我们就可以在运行期将eoc_myLowercaseString的选择子对应原有的lowercaseString方法。通过以下代码来交换这两个方法的实现：
```
Method originalMethod = class_getInstanceMethod([NSString class], @selector(lowercaseString));

Method swappedMethod = class_getInstanceMethod([NSString class], @selector(eoc_myLowercaseString));

method_exchangeImplementations(originalMethod, swappedMethod);
```
在调用的时候，就可以即刻看到Log：
```
NSString *string = "StRiNg";
NSString *lowercaseString = [string lowercaseString];
//Output: StRiNg => string
```
通过此方案，开发者可以为那些“完全不知道其具体实现的”黑盒方法添加日志记录功能，非常有助于程序调试。然而，此方法只在调试程序时有用，很少有人在调试程序之外用该方法来永久改动某个类的功能，如果滥用，会令代码变得不易读懂且难以维护。
