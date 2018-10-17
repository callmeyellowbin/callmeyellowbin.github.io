---
layout:     post
title:      学习Foundation Kit
#subtitle:   学习如何使用复合
date:       2018-05-23
author:     Hoben Wong
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - iOS
    - Objective-C
---
Cocoa分为两种框架：Foundation Kit和Application Kit。Application Kit包含了所有的用户接口对象和高级类，今天学习的Foundation Kit则有很多有用的、面向数据的低级类和数据类型，如NSString、NSArray、NSEnumerator、NSNumber。
###一.String
####1.NSString
- stringWithFormat
  通过格式字符串和参数来创建NSString，用法：
```
NSString *height;
height = [NSString stringWithFormat:@"Your height is %d feet, %d inches", 5, 11];
NSLog(@"%@", height);
```
注意这个NSLog，如果是**NSLog(height)**的话会有警告:Format string is not a string literal (potentially insecure)，因为Compiler认为你并不懂NSLog的真正用法。而**NSLog(@"%@", height)**才是正确的用法。
- length
  获取NSString的长度大小。用法：
```
if ([height length] > 20)
     NSLog(@"Tall!");
```
- 比较
  比较是否相等，用isEqualToString，不要用==！
```
//上面这种方法可以判断字符串内容相等
if ([height isEqualToString: another])
    NSLog(@"isEqual!");
//下面这种方法不能判断
if (height == another)
    NSLog(@"Memory is Equal!");
```
比较包含关系，有以下两种方法：
\- (BOOL) hasPrefix: (NSString *) aString;  //用于判断前序包含
\- (BOOL) hasSuffix: (NSString *) aString;  //用于判断后序包含
```
NSString *filename;
filename = [NSString stringWithFormat: @"jp.avi"];
if ([filename hasSuffix: @".avi"])
    NSLog(@"It is an .avi!");
else if ([filename hasSuffix: @".mp3"])
    NSLog(@"It is an .mp3!");
```
还有更屌的，寻找子字符串的开始位置和长度，用NSRange的rangeOfString方法，location是起始位置，length是长度。
```
NSString *filename;
filename = [NSString stringWithFormat: @"jp-activity-classroom.avi"];
NSRange range;
range = [filename rangeOfString: @"classroom"];
NSLog(@"%s start from %d, the length is %d", "classroom",
    (int)range.location,
    (int)range.length);
```
####2.NSMutableString
事实上，NSString是不可变的，其子类NSMutableString才可以直接进行删除、添加等操作（类似于StringBuffer）。
- 创建：
[NSMutableString  stringWithCapacity: (int) capacity]:创建一个capacity大小的string（预先分配好内存），但是超过了也不要紧，操作速度会慢一点。
[string appendString: @(NSString *) str]：接受参数str，然后将其复制到接收对象的末尾。
[string appendFormat: @(NSString *) str]：和上面的类似，但它不会创建新的字符串对象，而是直接将格式化的字符串附加在接受字符串的末尾。
```
NSMutableString *string;
string = [NSMutableString stringWithCapacity: 50];
[string appendString: @"James Hoben Eric Road JP "];
[string appendFormat: @"WhoWhenHow"];
NSLog(@"%@", string);
```
- 删除操作
  删除要靠deleteCharactersInRange和rangeOfString相结合，首先通过rangeOfString获得要删除的起始点和长度，再用deleteCharactersInRange函数搞定。
```
NSRange range = [string rangeOfString: @"JP"];
range.length++;  //删除多一个空格
[string deleteCharactersInRange: range];
```
- 获得子串
  子串要用substringWithRange获得，返回类型为NSString。
```
NSString *subString;
NSRange subRange;
//方法1
NSRange subRange = NSMakeRange(0, 5);
//方法2
//subRange.location = 0;
//subRange.length = 5;
subString = [string substringWithRange: subRange];
```
###二.Array
####1.NSArray
NSArray可以存放任意类型的对象，如NSString、Car、Shape、Tire等等（但不包括C语言的基本数据类型，如int、float、enum、struct，也不能存放nil）。
- 创建与访问
  NSArray的创建是[NSArray arrayWithObjects: ...]，访问数量是[array count]，访问内容则是直接array[index]，下面举个例子：
```
NSArray *array;
array = [NSArray arrayWithObjects: @"jp", @"Hoben", @"Eric", @"Road", nil];
for (int i = 0; i < [array count]; i++) {
    NSLog(@"index %d is %@", i, array[i]);
}
```
今天回顾了一下，找了一篇很不错的博客，补充点内容：
https://www.jianshu.com/p/c8caa30afd9d
- 访问特定元素的下标：
```
NSLog(@"The index of %s is %d", "jp", (int)[array indexOfObject: @"jp"]);
```
- 判断是否存在一个元素
```
if ([array containsObject: @"Taylor"])
    NSLog(@"Wow! Taylor is here!");
else
    NSLog(@"Unfortunately, only Diaosi here");
```
- 简化创建
  早说啊，还有这种创建方式，根本就不用那么麻烦。。
```
NSArray *array;
array = @[@"jp", @"Hoben", @"Eric", @"Road"];
```
####2.NSMutableArray
NSMutableArray同样是可变数组，用于添加和删除操作。
- 创建、添加与删除
  同样需要用[NSMutableArray arrayWithCapacity: (int) capacity]初始化。
  从NSArray引入用addObjectsFromArray，加入对象用addObject。
```
NSMutableArray *mutableArray;
mutableArray = [NSMutableArray arrayWithCapacity: 17];
[mutableArray addObjectsFromArray: array];
[mutableArray addObject: @"WhoWhenHow"];
```
删除有两种方法，第一种是根据下标删除，第二种是根据内容删除（值得注意的是，如果要删除的内容在原Array不存在的话，那么将不会进行任何操作，也没有异常抛出）。
```
NSMutableArray *mutableArray;
[mutableArray removeObjectAtIndex: 0];
[mutableArray removeObject: @"Road"];
[mutableArray removeObject: @"Null"];
```
###三.枚举
枚举分为索引枚举（用for）、枚举器和快速枚举，这里介绍一下快速枚举：
```
for (NSString *string in array) {
    NSLog(@"I found %@", string);
}
```
没错，就是一个语法和Python差不多的枚举。
###四.Dictionary
####1.NSDictionary
每个语言都有自己的键值对存储功能，OC的键值对功能就是靠NSDictionary实现的。
NSDictionary存储的方式为[NSDictionary  dictionaryWithObjectsAndKeys: (id) object, (id) key, ..., nil]。
取值的方式为(id) object = dictionary[key];
再以轮子哥为例子：
```
Tire *t1 = [Tire new];
Tire *t2 = [Tire new];
Tire *t3 = [Tire new];
Tire *t4 = [Tire new];
//方法一
NSDictionary *tires = [NSDictionary dictionaryWithObjectsAndKeys:
                       t1, @"front-letf",
                       t2, @"front-right",
                       t3, @"back-left",
                       t4, @"back-right"
                       , nil];
//方法二，快速初始化
NSDictionary *tires = @{@"front-left": t1,
                             @"front-right": t2,
                             @"back-left": t3,
                             @"back-right": t4
                             };
NSLog(@"the front-right tire is %@", tires[@"front-right"]);
```
####2.NSMutableDictionary
没错，总有一对cp出现，他就是Mutable！
具体已经不太想说了，直接上例子吧，我看得很晕。
值得注意的是setObject: (id) object forkey: (id) key这个方法。
```
NSMutableDictionary *mutableTires;
mutableTires = [NSMutableDictionary dictionaryWithCapacity: 20];
[mutableTires setDictionary: tires];
[mutableTires setObject:t1 forKey:@"front-left"];  //设置
[mutableTires removeObjectForKey: @"front-right"];  //删除
NSLog(@"%@", mutableTires[@"front-right"]);  //输出null 
NSLog(@"%@", mutableTires[@"front-left"]);  //有输出
[mutableTires setObject: t2 forKey:@"front-right"];  //新增
NSLog(@"%@", mutableTires[@"front-right"]);  //有输出
```
也就是说，如果原dictionary里有这个key的话，就会将对应的value值更新，否则，将会新建一个key-value对。
###五.NSNumber
NSArray和NSDictionary不能存储基本类型的数据，这可很麻烦。没关系，Cocoa提供了NSNumber来包装，可用于封装int、char、float、bool四大数据类型。
下面展示一下怎么把它们放到NSArray中：
```
NSNumber *number;
number = [NSNumber numberWithInt: 42];
NSArray *array;
array = [NSArray arrayWithObjects: number, nil];
NSLog(@"%@", array[0]);
```
- NSNumber的提取
```
NSNumber *numberInt, *numberFloat;
numberInt = [NSNumber numberWithInt: 42];
NSMutableArray *array;
array = [NSMutableArray arrayWithObject: numberInt];
numberFloat = [NSNumber numberWithFloat: 1.2];
[array addObject: numberFloat];  //放入一个42和一个1.2
//[NSNumber intValue]即以int类型取出
float result = [array[0] intValue] + [array[1] floatValue]; 
```
###六.CGPoint、CGRect、CGSize和NSValue
####1.CGPoint
CGPoint内包含两个float类型的x和y，对应着一个坐标。
```
CGPoint point = CGPointMake(2, 3);
NSLog(@"%d, %d", (int) point.x, (int) point.y);
```
####2.CGSize
CGSize内包含float类型的width和height，对应着宽和高。
```
CGSize size = CGSizeMake(4, 5);
NSLog(@"%d, %d", (int) size.width, (int) size.height);
```
####3.CGRect
CGRect内包含CGPoint和CGSize，再细分就是x, y, width, height。
```
CGRect rect = CGRectMake(point.x, point.y, size.width, size.height);
NSLog(@"%d, %d, %d, %d",
      (int) rect.origin.x,
      (int) rect.origin.y,
      (int) rect.size.width,
      (int) rect.size.height);
```
####4.NSValue —— 自定义类的容器
事实上，上文提到的NSNumber就是继承自NSValue的，NSValue可以提供对任意类型的类的封装。
- 创建
```
NSValue *valuePoint, *valueSize, *valueRect, *valueTire;
valuePoint = [NSValue valueWithCGPoint: point];
valueSize = [NSValue valueWithCGSize: size];
valueRect = [NSValue valueWithCGRect: rect];
//下面是自定义的类，注意变量名前面加&
valueTire = [NSValue value: &tire withObjCType: @encode(Tire)];
```
- 放入NSArray
```
array = @[valuePoint, valueSize, valueRect, valueTire];
```
- 枚举与判断类型
  重点说下如何判断NSValue里面的类型，NSValue里面的objCType是char*类型，因此我们需要用strcmp()方法去比较[value objCType]和@encode(Your Class)是否相等。
```
for (NSValue *value in array) {
    if (strcmp([value objCType], @encode(CGPoint)) == 0)
        NSLog(@"%d, %d", (int) point.x, (int) point.y);
    if (strcmp([value objCType], @encode(CGSize)) == 0)
        NSLog(@"%d, %d", (int) size.width, (int) size.height);
    if (strcmp([value objCType], @encode(CGRect)) == 0)
        NSLog(@"%d, %d, %d, %d",
              (int) rect.origin.x,
              (int) rect.origin.y,
              (int) rect.size.width,
              (int) rect.size.height);
    if (strcmp([value objCType], @encode(Tire)) == 0)
        NSLog(@"It is a tire!");
}
```
###七.NSNull
之前说过NSArray和NSDictionary都不能存放nil，如果类型里面真的有空怎么办呢？这时候就要用到NSNull了。
```
NSDictionary *dictionary;
dictionary = @{@"id": [NSNull null], @"name": @"Hoben Wong", @"Tel": @123456};
if (dictionary[@"id"] == [NSNull null])
    NSLog(@"%@'s id is empty!", dictionary[@"name"]);
```
