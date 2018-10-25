---
layout:     post
author:     Hoben Wong
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - 前端学习笔记
    - JavaScript学习笔记
---

### 一. JavaScript的实现

- ECMAScript：描述了该语言的语法和基本对象
- DOM：描述了处理网页内容的方法和接口
- BOM：描述了与浏览器进行交互的方法和接口

### 二. ECMAScript语法
语法基础与Java类似，只挑一些特别的点来写。

#### 1. var
定义变量只用`var`运算符，可以初始化为任意值，因此，可以随时改变变量所存储的数据类型。

#### 2. 函数
基本语法：

```
function func(arg0, arg1, ... argN) {
    statements
}
```

#### 3. arguments对象
使用特殊对象`arguments`，开发者无需明确指出参数名，就能访问它们。

```
function sayHi() {
    if (arguments[0] == "bye") {
        return;
    }
    alert(arguments[0]);
}
```

也可以检测参数的个数：

```
function howManyArgs() {
    alert(arguments.length);
}

howManyArgs("string", 45);    //2
howManyArgs();                //0
howManyArgs("string");        //1
```

还可以模拟函数重载：

```
function doAdd() {
    if (arguments.length == 1) {
        alert(arguments[0] + 5);
    } else if (arguments.length == 2) {
        alert(arguments[0] + arguments[1]);
    }
}
```

#### 4. 函数闭包
在ECMAScript中，函数也是功能完整的对象，就是说，我们可以把函数当作参数来传入函数中，也可以返回一个函数。

```
function callAnotherFunc(fnFunction, vArgument) {
    fnFunction(vArgument);
}

var doAdd = new Function("iNum", "alert(iNum + 10)");

callAnotherFunc(doAdd, 10);    //20

alert(callAnotherFunc.length);	//输出 "2"
```

闭包，即函数可以使用函数之外定义的常量。

```
var sMessage = "hello world";

function sayHelloWorld() {
  alert(sMessage);
}

sayHelloWorld();
```

如果在一个函数中定义另一个会使得闭包更加复杂：

```
var iBaseNum = 10;

function addNum(iNum1, iNum2) {
    function doAdd() {
        return iNum1 + iNum2 + iBaseNum;
    }
    return doAdd();
}
```

doAdd()函数根本不接受参数，它使用的值是从执行环境中获取的。

#### 5. 作用域
ECMAScript只存在公用作用域，其所有对象的所有属性和方法都是公用的。因此，定义自己的类和对象时，必须格外小心。开发者确定了一个规约，说明哪些属性和方法应该被看做私有的，会在前面加下划线（注意，下划线并不改变属性是公用属性的事实）：

 ```
obj._color_ = "blue";
 ```

静态作用域定义的属性和方法任何时候都能从同一位置访问，在Java中，类可具有属性和方法，无需实例化该类的对象，即可访问这些属性和方法。

ECMAScript没有静态作用域，但是他可以给构造函数提供属性和方法。

```
function sayHello() {
    alert("Hello");
}

sayHello.alternate = function() {
    alert("hi");
}

sayHello();    //"hello"

sayHello.alternate();    //"hi"
```

在ECMAScript中，关键字`this`总是指向调用该方法的对象，如：

```
var oCar = new Object();
oCar.color = "red";
oCar.showColor = function() {
    alert(this.color);
    //和alert(oCar.color)等价
}

oCar.showColor();    //"red"
```

#### 5. 创建对象的方法
- 工厂方式

```
function showColor() {
    alert(this.color);
}

function createCar(sColor, iDoors, iMpg) {
    var temp = new Object();
    temp.color = sColor;
    temp.doors = iDoors;
    temp.mpg = iMpg;
    temp.showColor = showColor;
    return temp;
}

var oCar = createCar("red", 4, 23);
oCar.showColor();
```

在createCar()内部，赋予对象一个指向已经存在的`showColor()`函数的指针。从功能上讲，这样解决了重复创建函数对象的问题；但从语义上讲，该函数不太像是对象的方法。

- 构造函数方式

```
function Car(sColor, iDoors, iMpg) {
    this.color = sColor;
    this.doors = iDoors;
    this.mpg = iMpg;
    this.showColor = function() {
        alert(this.color);
    }
}

var oCar1 = new Car("red", 4, 23);
oCar.showColor();
```

但是以上方法的构造函数会重复生成函数，为每个对象都创建独立的函数版本。当然也可以写成工厂方式的方法构造，但是这样做也是没有语义的。

- 原型方式
  该方式利用了对象的`prototype`属性，可以把它看成创建新对象所依赖的原型。

```
function Car() {
}

Car. prototype.color = "blue";
Car. prototype.doors = 4;
Car. prototype.mpg = 25;
Car. prototype.showColor = function() {
    alert(this.color);
};

var oCar1 = new Car();
```

调用new Car()时，原型的所有属性都被立即赋予要创建的对象，意味着所有`Car`实例存放的都是指向`showColor()`函数的指针。从语义上讲，所有属性看起来都属于一个对象，因此解决了前面两种方式存在的问题。

但是，如果使用数组，会产生对象共享的问题，比如car1改变了Car()里面的数组之后，car2也会受到影响。

- 混合的构造函数/原型方式
  用构造函数定义对象的所有非函数属性，用原型方式定义对象的函数属性。结果：所有函数只出现一次，每个对象都具有自己的对象属性实例。

```
function Car(sColor, iDoors, iMpg) {
    this.color = sColor;
    this.doors = iDoors;
    this.mpg = iMpg;
    this.drivers = new Array("Mike", "John");
}

Car.prototype.showColor = function() {
    alert(this.color);
}

var oCar1 = new Car("red", 4, 23);
var oCar2 = new Car("blue", 3, 25);

oCar1.drivers.push("Bill");

alert(oCar1.drivers);    //输出 "Mike,John,Bill"
alert(oCar2.drivers);    //输出 "Mike,John"
```

- 动态原型方法
  和上一种方法的唯一区别是赋予对象方法的位置。

```
function Car(sColor, iDoors, iMpg) {
    this.color = sColor;
    this.doors = iDoors;
    this.mpg = iMpg;
    this.drivers = new Array("Mike", "John");

    if (typeof Car._initialized == "undefined") {
        Car.prototype.showColor = function() {
            alert(this.color);
        };

        Car._initialized = true;
    }
}
```

#### 6. 修改对象
重命名已有方法：

```
Array.prototype.enqueue = function(vItem) {
    this.push(vItem);
}

Array.prototype.dequeue = function() {
    return this.shift();
}
```

添加新方法：

```
Array.prototype.indexOf = function(vItem) {
    for (var i=0; i<this.length; i++) {
        if (vItem == this[i]) {
	      return i;
	    }
     }

    return -1;
}
```

重定义已有方法：

```
Function.prototype.toString = function() {
    return "It is hidden.";
}
```

存储原方法的指针，以便以后的使用：

```
Function.prototype.originalToString = Function.prototype.toString;

Function.prototype.toString = function() {
    if (this.originalToString().length > 100) {
        return "Too long to display.";
    } else {
        return this.originalToString();
    }
};
```

#### 7. 继承
- 对象冒充

```
function ClassA(sColor) {
    this.color = sColor;
    this.sayColor = function() {
        alert(this.color);
    };
}

function ClassB(sColor, sName) {
    this.newMethod = ClassA;
    this.newMethod(sColor);
    delete this.newMethod;

    this.name = sName;
    this.sayName = function() {
        alert(this.name);
    };
}
```

在上面的代码中，为`ClassA`赋予了方法`newMethod`（函数名只是指向它的指针）。然后调用该方法，传递给它的是`ClassB构造函数`的参数`sColor`，最后要删除对`ClassA`的引用，这样以后就不能再调用它。

- 对象冒充实现多重继承

```
function ClassZ() {
    this.newMethod = ClassX;
    this.newMethod();
    delete this.newMethod;

    this.newMethod = ClassY;
    this.newMethod();
    delete this.newMethod;
}
```

小弊端：如果ClassX和ClassY具有同名的属性或方法，则ClassY具有高优先级。

- call()方法
  与经典的对象冒充最相似的方法。它的第一个参数用作this的对象，其他参数都直接传递给函数自身。如：

```
function sayColor(sPrefix, sSuffix) {
    alert(sPrefix + this.color + sSuffix);
}

var obj = new Object();
obj.color = "blue";

sayColor.call(obj, "The color is ", " nice color!");
```

套用到对象冒充方法，则这样写：

```
function ClassB(sColor, sName) {
   // this.newMethod = ClassA;
   // this.newMethod(sColor);
   // delete this.newMethod;
    
    ClassA.call(this, sColor);

    this.name = sName;
    this.sayName = function() {
        alert(this.name);
    };
}
```

- apply()方法
  apply()方法有两个参数，用作this的对象和要传递给函数的参数的数组。同理可以套用到对象冒充的方法中：

```
function ClassB(sColor, sName) {
   // this.newMethod = ClassA;
   // this.newMethod(sColor);
   // delete this.newMethod;
    
    ClassA.apply(this, new Array(sColor));

    this.name = sName;
    this.sayName = function() {
        alert(this.name);
    };
}
```

- 原型链

```
function ClassA() {
}

ClassA.prototype.color = "blue";
ClassA.prototype.sayColor = function() {
    alert(this.color);
};

function ClassB() {
}

ClassB.prototype = new ClassA();

ClassB.prototype.name = "";
ClassB.prototype.sayName = function() {
    alert(this.name);
};

var objA = new ClassA();
var objB = new ClassB();
objA.color = "blue";
objB.color = "red";
objB.name = "John";
objA.sayColor();
objB.sayColor();
objB.sayName();
```

把`ClassB`的`prototype`属性设置为`ClassA`的实例。缺点：不能支持多重继承。

- 混合方式
  对象冒充的主要问题是必须使用构造函数方式，使用原型链则无法使用带参数的构造函数了，这时候需要我们两者都使用。

```
function ClassA(sColor) {
    this.color = sColor;
}

ClassA.prototype.sayColor = function() {
    alert(this.color);
};

function ClassB(sColor, sName) {
    ClassA.call(this, sColor);
    this.name = sName;
}

ClassB.prototype = new ClassA();

ClassB.prototype.sayName = function() {
    alert(this.name);
};
```
