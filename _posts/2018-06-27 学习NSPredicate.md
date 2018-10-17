###一.概念
Cocoa提供一个名为`NSPredicate`的类，用于指定过滤器的条件。可以创建这个对象，通过该对象准确地描述所需的文件，对每个对象通过谓词进行筛选，判断它们是否与条件相匹配。
###二.相关样例
```
Car *car1 = [[Car alloc] init];
[car1 setName: @"Hoben"];
[car1 setHorsePower: 160];

NSPredicate *predicate = [NSPredicate predicateWithFormat: @"name == 'Hoben'"];
BOOL match = [predicate evaluateWithObject: car1];
NSLog(@"%d", match);
```
```
NSPredicate *predicate = [NSPredicate predicateWithFormat: @"name == 'Hoben'"];
BOOL match = [predicate evaluateWithObject: car1];
NSLog(@"%d", match);

Car *car2 = [[Car alloc] init];
[car2 setName: @"JP"];
[car2 setHorsePower: 100];

Car *car3 = [[Car alloc] init];
[car3 setName: @"Road"];
[car3 setHorsePower: 200];

NSArray *cars = [[NSArray alloc] initWithObjects: car1, car2, car3, nil];

predicate = [NSPredicate predicateWithFormat: @"horsePower > 150"];
for (Car *car in cars) {
    BOOL matchPower = [predicate evaluateWithObject: car];
    if (matchPower)
        NSLog(@"%@", [car name]);
}
```
用另一个函数`filteredArrayUsingPredicate`替代for循环：
```
NSArray *results;
results = [cars filteredArrayUsingPredicate: predicate];
NSLog(@"%@", results);
```
以下两句等价：
```
predicate = [NSPredicate predicateWithFormat: @"horsePower > 150 && horsePower < 170"];
predicate = [NSPredicate predicateWithFormat: @"horsePower BETWEEN {150, 170}"];
```
使用IN：
```
predicate = [NSPredicate predicateWithFormat: @"Self.name IN {'Hoben', 'JP'}"];
```
####字符串运算符
字符串运算符中，可以使用`BEGINSWITH`,`CONTAINS`,`ENDSWITH`，后加修饰符[cd]/[c]/[d]，分别代表：不区分大小写和发音、不区分大小写、不区分发音
```
predicate = [NSPredicate predicateWithFormat: @"Self.name BEGINSWITH[cd] 'Ho'"];
predicate = [NSPredicate predicateWithFormat: @"Self.name CONTAINS[c] 'ho'"];
predicate = [NSPredicate predicateWithFormat: @"Self.name ENDSWITH 'ben'"];
```
可以运用`LIKE`运算符，`*ob*`代表包含"ob"的字符串，`*??be*`代表`be`前面至少有两个字符。
```
predicate = [NSPredicate predicateWithFormat: @"Self.name LIKE '*ob*'"];
predicate = [NSPredicate predicateWithFormat: @"Self.name LIKE '*??be*'"];
```