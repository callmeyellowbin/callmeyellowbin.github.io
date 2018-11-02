---
layout:     post
author:     Hoben Wong
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - 前端学习笔记
    - JavaScript学习笔记
---

### 一. 处理HTML内容
#### 1. 写入HTML输出：

```
<!DOCTYPE html>
<html>
<body>

<script>
document.write("<h1>This is a heading</h1>");
document.write("<p>This is a paragraph.</p>");
</script>

</body>
</html>
```

#### 2.对事件做出反应：

```
<button type = "button" onclick = "alert('Welcome!')">点击这里</button>
```

#### 3. 改变HTML内容：

```
<!DOCTYPE html>
<html>
<body>

<p id="demo">
JavaScript 能改变 HTML 元素的内容。
</p>

<script>
function myFunction() {
    x = document.getElementById("demo");    //找到元素
    x.innerHTML = "Hello JavaScript!";      //改变内容
}
</script>

<button type="button" onclick="myFunction()">点击这里</button>

</body>
</html>
```

#### 4. 改变HTML图像：

```
<!DOCTYPE html>
<html>
<body>
<script>
function changeImage() {
    element = document.getElementById('myimage');
    if (element.src.match("bulbon")) {
        element.src = "/i/eg_bulboff.gif";
    } else {
        element.src = "/I/eg_bulbon.gif";
    }
}
</script>

<img id="myimage" onclick="changeImage()" src="/i/eg_bulboff.gif">

<p>点击灯泡来点亮或熄灭这盏灯</p>

</body>
</html>
```

#### 5. 改变HTML元素的样式：

```
x = document.getElementById("demo");
x.style.color = "#ff0000";
```

通常情况下，我们会把js函数放入\<head\>部分中，或者放在页面底部。这样就可以把它们安置到同一处位置，不会干扰页面的内容。

```
<!DOCTYPE html>
<html>
<head>
<script>
function myFunc() {
    element = document.getElementById('demo');
    element.innerHTML = "Hello World!";
}
</script>
</head>

<body>
<p id = "demo">Click to show</p>
<button type = "button" onclick = "myFunc()">Try it</button>
</body>
</html>
```

#### 6. 写到文档输出

```
<!DOCTYPE html>
<html>
<body>

<h1>我的第一张网页</h1>

<script>
document.write("<p>我的第一段JavaScript</p>");
</script>

</body>
</html>
```

但是要注意的是，`document.write()`仅仅向文档输出写内容。如果在文档已完成加载后执行该函数，则会覆盖整个HTML页面。

```
<!DOCTYPE html>
<html>
<body>

<h1>My First Web Page</h1>

<p>My First Paragraph.</p>

<button onclick="myFunction()">点击这里</button>

<script>
function myFunction()
{
document.write("糟糕！文档消失了。");
}
</script>

</body>
</html>
```

#### 7. 捕获异常：

```
<script>
function message() {
    try {
        adddddd("");
    } catch (err) {
        alert("Error!");
    }
}
</script>
```

#### 8. 验证表单：

```
<html>
<head>
<script type = "text/javascript">
function validate_required(field, alerttxt) {
    with (field) {
        if (value == null || value == "") {
            alert(alerttxt);
            return false;
        } else {
            return true;
        }
    }
}

function validate_form(thisform) {
    with (thisform) {
        if (validate_required(email, "Email must be filled out!") == false) {
            email.focus();
            return false;
        }
    }
}
</script>
</head>

<body>
<form action = "submitpage.htm" onsubmit = "return validate_form(this)" method = "post">
Email:<input type = "text" name = "email" size = "30">
<input type = "submit" value = "Submit">
</form>
</body>
```

### 二.JavaScript + DOM

当网页被加载时，浏览器会创建页面的文档对象模型（Document Object Model）。HTML的DOM模型被构造为对象的树。

![](https://upload-images.jianshu.io/upload_images/8407639-414c0a6f1be15385.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过DOM，JavaScript获得了足够的能力来创建动态的HTML。

- JavaScript能够改变页面中的所有HTML元素
- JavaScript能够改变页面中的所有HTML属性
- JavaScript能够改变页面中的所有CSS样式
- JavaScript能够对页面中的所有事件作出反应

#### 1. 通过id查找HTML元素

```
var x = document.getElementById("intro");
```

#### 2. 通过标签名查找HTML元素
查找id="main"的所有\<p\>元素：

```
var x = document.getElementById("main");
var y = x.getElementByTag("p");
```

#### 3. 改变HTML内容

```
document.getElementById("id").innerHTML = "New Text";
```

#### 4. 改变HTML属性

```
document.getElementById("image").src = "newImage.jpg";
```

#### 5. 改变HTML样式

```
document.getElementById("id").style.color = "blue";
```

```
<!DOCTYPE html>
<html>
<body>

<p id = "p1"> 文本 </p>

<input type = "button" value = "隐藏文本" onclick = "document.getElementById('p1').style.visibility = 'hidden'"/>

<input type = "button" value = "显示文本" onclick = "document.getElementById('p1').style.visibility = 'visible'"/>

</body>
</html>
```

也可以通过JavaScript来分配onclick事件

```
<script>
document.getElementById("myBtn").onClick = function() {displayDate()};
</script>
```

#### 6. onload和onunload事件
这两个事件会在用户进入或者离开页面时被触发。可以用于检测访问者的浏览器类型和浏览器版本，并基于这些信息来加载网页的正确版本。可用于处理cookie。

```
<body onload = "checkCookies()">
```

#### 7. onchange事件
当用户改变输入字段的内容时会调用。

```
<input type = "text" id = "fname" onchange = "upperCase()">
```

#### 8. onmouseover和onmouseout事件
处理用户移至HTML元素上方或移出元素时触发函数。

```
<!DOCTYPE html>

<head>

<style>
#mouse {
    background-color: green;
    width: 120px;
    height: 20px;
    padding: 40px;
    color: #ffffff;
}
</style>

<script>
function mOver(obj) {
    obj.innerHTML = "谢谢";
    obj.style.backgroundColor = "black";
}
function mOut(obj) {
    obj.innerHTML = "把鼠标移到上面";
    obj.style.backgroundColor = "green";
}
</script>

</head>

<html>

<body>

<div id = "mouse" onmouseover = "mOver(this)" onmouseout = "mOut(this)" >把鼠标移到上面</div>

</body>
</html>
```

注意：在CSS中为`background-color: green`，而在JavaScript中为`backgroundColor = "green"`。

#### 8. 创建元素
如需向HTML DOM添加新元素，必须首先创建该元素（元素节点），然后向一个已存在的元素追加该元素。

```
<div id = "div1">
<p id = "p1"> 这是一个段落 </p>
<p id = "p2"> 这是一个段落 </p>
</div>

<script>
var para = document.createElement("p");
var node = document.createTextNode("这是新段落");
para.appendChild(node);

var element = document.getElementById("div1");
element.appendChild(para);
</script>
```

#### 9. 删除元素
```
<div id = "div1">
<p id = "p1"> 这是一个段落 </p>
<p id = "p2"> 这是一个段落 </p>
</div>

<script>
var parent = document.getElementById("div1");
var child = document.getElementById("p2");
parent.removeChild(child);
</script>
```

#### 10. 替换元素
```
<div id = "div1">
<p id = "p1"> 这是一个段落 </p>
<p id = "p2"> 这是一个段落 </p>
</div>

<script>
var para = document.createElement("p");
var node = document.createTextNode("This is new");
para.appendChild(node);

var parent = document.getElementById("div1");
var child = document.getElementById("p2");
parent.replaceChild(para, child);
</script>
```

#### 11. 常用属性
- innerHTML - 节点（元素）的文本值
- parentNode - 节点（元素）的父节点
- childNodes - 节点（元素）的子节点
- attributes - 节点（元素）的属性节点

##### nodeName属性
- nodeName是只读的
- 元素节点的nodeName与标签名相同
- 属性节点的nodeName与属性名相同
- 文本节点的nodeName始终是\#text
- 文档节点的nodeName始终是\#document

##### nodeValue属性
- 元素节点的nodeValue是undefined或者null
- 文本节点的nodeValue是文本本身
- 属性节点的nodeValue是属性值

下面的代码会输出两行Hello World！

```
<html>
<body>
<p id = "intro"> Hello World! </p>

<script type = "text/javascript">
x = document.getElementById("intro");
document.write(x.firstChild.nodeValue);
</script>

</body>
</html>
```

### 三. jQuery
jQuery是一个JavaScript函数库，可以通过一行简单的标记被添加到网页中，包含以下特性：
- HTML元素选取
- HTML元素操作
- CSS操作
- HTML事件函数
- JavaScript特效和动画
- HTML DOM遍历和修改
- AJAX
- Utilities

#### 1. 语法
基础语法：\$(selector).action()
- 美元符号定义jQuery
- 选择符查询HTML元素
- jQuery的action（）执行对元素的操作。

下面的代码是文档就绪函数，所有的jQuery函数都会位于这个document ready函数里面，这是为了防止文档在完全加载之前运行jQuery代码，因为如果在文档没有完全加载之前就运行函数，操作可能失败。如：

- 试图隐藏一个不存在的元素
- 获得未完全加载的图像的大小

```
$(document).ready(function() {
    // jQuery function
});
```

##### a. 选择器
元素选择器：

```
$("p")    //选取<p>元素
$("p.intro")    //选取所有 class = "intro" 的<p>元素
$("p#demo")    //选取所有 id = "demo" 的<p>元素
```

属性选择器：

```
$("[href]")    //选取所有带有href属性的元素
$("[href = '#']")    //选取所有带有href值等于“#”的元素
$("[href $= '.jpg']")    //选取所有href值以".jpg"结尾的元素
```

CSS选择器：

```
//把所有p元素的背景颜色更改为红色
$("p").css("background-color", "red");
```

##### b. 事件函数
当HTML中发生某些事件时所调用的方法。

```
<html>
<head>
<script  type = "text/javascript" src = "jquery.js"></script>
<script type = "text/javascript">
$(document).ready(function() {
    $("button").click(function() {
        $("p").hide("slow");
    });
});
</script>
</head>

<body>
<h2>heading</h2>
<p>I will disappear</p>
<p>I will disappear, too</p>
<button>click me</button>
</body>

</html>
```

![](https://upload-images.jianshu.io/upload_images/8407639-801bf3bb92ea81c0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果使用`toggle()`则可以切换`hide()`和`show()`方法。两个可选参数speed和callback，分别代表着规定隐藏/显示的速度、toggle()完成后执行的函数名称。

```
$("button").click(function() {
    $("p").toggle(speed, callback);
});
```

##### c. 各种效果
jQuery有四种fade方法：

- fadeIn()
- fadeOut()
- fadeToggle()
- fadeTo()

jQuery有三种滑动方法：

- slideDown()
- slideUp()
- slideToggle()

创建自定义动画方法为`animate()`

```
$(selector).animate({params}, speed, callback);
```

如：把<div>元素移动到左边，直到`left属性`等于250px为止，也可以设定属性的动画值：

```
$("button").click(function() {
    $("div").animate({left: '250px', height: 'toggle'});
});
```

如果需要设定多个动画来先后展示的话，也可以使用队列功能：

```
$("button").click(function() {
    var div = $("div");
    $("div").animate({height: '300px', opacity: '0.4'}, "slow");
    $("div").animate({width: '300px', opacity: '0.8'}, "slow");
    $("div").animate({height: '100px', opacity: '0.4'}, "slow");
    $("div").animate({width: '100px', opacity: '0.8'}, "slow");    
});
```

##### d. Callback
Callback函数会在当前动画完全执行之后再执行：

```
$("p").hide(1000, function() {
    alert("The paragraph is now hidden");
});
```

##### e. Chaining
可以使用链式语法来运行jQuery命令。

```
$("#p1").css("color", "red")
        .slideUp(2000)
        .slideDown(2000);
```

#### 2. 获得属性和内容

##### a.获取内容
三个用于DOM操作的jQuery方法：

text()：设置或返回所选元素的`文本内容`

```
$("#btn1").click(function() {
    alert($("#test").text());
});
```

html()：设置或返回所选元素的`内容（包括HTML标记）`

```
$("#btn2").click(function() {
    alert($("#test").html());
});
```

val()：设置或返回表单字段的值

```
$("#btn3").click(function() {
    alert($("#test").val());
});
```

attr()：设置或返回属性值

```
$("#btn4").click(function() {
    alert($("#test").attr("href"));
});
```

##### b. 改变内容

```
$("#btn1").click(function() {
    $("#test1").text("Hello World!");
});

$("#btn2").click(function() {
    $("#test2").html("<b>Hello World!</b>");
});

$("#btn3").click(function() {
    $("#test3").val("HHH");
});

$("#btn4").click(function() {
    $("#test3").attr("href", "http://www.baidu.com");
});
```

回调函数：有两个参数，分别是被选元素列表中当前元素的下标，以及原始值。然后以函数新值返回希望使用的字符串：

```
$("#btn1").click(function() {
    $("#test1").text(function(i, origText) {
        return "Old text: " + origText + " New text: Hello world! Index: " + i;
    });
});

$("#btn2").click(function(){
  $("#test2").html(function(i,origText){
    return "Old html: " + origText + " New html: Hello <b>world!</b>
    (index: " + i + ")";
  });
});

$("#btn4").click(function() {
    $("#test3").attr("href", function(i, origValue) {
        return origValue + "/jquery";
    });
});
```

##### c. 添加元素

```
function appendText() {
    var txt1 = "<p>Text.</p>";    //以HTML创建新元素
    var txt2 = $("<p></p>").text("Text.");    //以jQuery创建新元素
    var txt3 = document.createElement("p");
    txt3.innerHTML = "Text.";
    var txt4 = document.createElement("p");
    var text = document.createTextNode("Text.");
    txt4.append(text);    //以DOM创建新元素
    $("body").append(txt1, txt2, txt3, txt4);
}
```

append()：在被选元素的**结尾**插入内容。
prepend()：在被选元素的**开头**插入内容。
after()：在被选元素**之后**插入内容。
before()：在被选元素**之前**插入内容。

append和after的区别：如果为一个`<span>元素`使用这两个函数，则append之后的元素还会在`\<span\>元素`里面，after之后的元素会在`<span>元素`后面。

##### d. 删除元素
remove()：删除被选元素（及其子元素）
empty()：从被选元素中删除子元素
remove(selector)：过滤被删除的元素

```
$("p").remove(".italic");
```

##### e. 获取并设置CSS类

addClass()：向被选元素添加一个或多个类
removeClass()：从被选元素删除一个或多个类
toggleClass()：切换
css()：设置或返回样式属性

```
<!DOCTYPE html>
<html>
<head>
<script src="/jquery/jquery-1.11.1.min.js"></script>
<script>
$(document).ready(function() {
    $("button").click(function() {
        $("h1, h2, p.parBlue").addClass(".blue");
        $("p.parImportant").addClass(".important");
    });
});
</script>

<style>
.blue {
    color: blue;
}

.important {
    font-weight: bold;
}
</style>
</head>

<body>
<h1>标题1</h1>
<h2>标题2</h2>
<p class = "parBlue">这是一个段落</p>
<p class = "parBlue">这是另一个段落</p>
<p class = "parImportant">这是非常重要的文本！</p>
<button>向元素添加类</button>
</body>

</html>
```

返回CSS属性：

```
$("p").css("color");
```

设置CSS属性：

```
$("p").css("color", "blue");
```

设置多个CSS属性：

```
$("p").css({"color": "blue", "font-size": "200%"});
```

#### 3. jQuery的遍历
##### a. 向上遍历DOM树
parent()：只向上一级DOM树进行遍历
parents()：返回被选元素的所有祖先，可以往里面添加过滤参数
parentsUntil()：返回介于两个给定元素的所有祖先元素（开区间）

它们的区别如下图所示：

![parent](https://upload-images.jianshu.io/upload_images/8407639-c2f322a294eac41b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![parents("ul")](https://upload-images.jianshu.io/upload_images/8407639-37d3e0f6bf06b6be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![parentsUntil("ul")](https://upload-images.jianshu.io/upload_images/8407639-ac9d1c3f7aabb3fc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### b. 向下遍历DOM树
children()：返回所有`直接子元素`
find()：返回被选元素的后代元素，一路向下直到最后一个后代

![children("span")](https://upload-images.jianshu.io/upload_images/8407639-7f87339bd1cecf87.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![find("span")](https://upload-images.jianshu.io/upload_images/8407639-089e80ec484273db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### c. 同胞
siblings()：返回被选元素的所有同胞元素
next()：返回下一个同胞元素
nextAll()：返回所有跟随的同胞元素
nextUntil()：返回介于两个给定参数之间的所有跟随同胞元素（开区间）
pre()：返回上一个同胞元素
preAll()：返回所有前面的同胞元素
preUntil()：返回介于两个给定参数之间的所有前面同胞元素（开区间）

##### d. 过滤
first()：首个元素
last()：尾元素
eq(index)：索引为index的元素
filter()：满足filter里面参数条件的元素
not()：和filter()相反

### 四. AJAX
AJAX：异步JavaScript和XML（Asynchronous JavaScript and XML），即不重载整个网页的情况下，AJAX通过后台加载数据，并在网页上进行显示。

#### 1. AJAX和XHR
XMLHttpRequest是AJAX的基础，用于在后台与服务器交换数据，这意味着可以在不重新加载整个网页的情况下，对网页的某部分进行更新。

创建XMLHttpRequest对象：

```
var xmlhttp;
if (window. XMLHttpRequest) {
    xmlhttp = new XMLHttpRequest;    //code for IE7+ and other
} else {
    xmlhttp = new ActiveXObject("Microsoft.XMLHTTP");    //code for IE5, IE6
}
```

#### 2. XHR请求
- open(method, url, async)：规定请求的类型、URL以及是否异步处理请求。
  method: 请求的类型（GET或者POST）
  url：文件在服务器上的位置
  async：true（异步）或false（同步）

- send(string)：将请求发送到服务器，加入的string仅用于POST请求。

GET请求实例：

```
<html>
<head>
<script>
function loadXML() {
   var xmlhttp;
   if (window.XMLHttpRequest) {
       xmlhttp = new XMLHttpRequest();
   } else {
       xmlhttp = new ActiveXObject("Microsoft.XMLHTTP");
   }
   xmlhttp.onreadystatechange = function() {
       if (xmlhttp.readyState == 4 && xmlhttp.status == 200) {
           document.getElementById("myDiv").innerHTML = xmlhttp.responseText;
       }
   }
   xmlhttp.open("GET", "/ajax/demo_get.asp?t=" + Math.random(), true);
   xmlhttp.send();
}

</script>
</head>

<body>
<h2>AJAX</h2>
<br/>
<button onclick = "loadXML()">请求数据</button>
<div id = "myDiv"></div>
</body>

</html>
```

POST请求实例如下，如果需要像HTML表单那样POST数据，则使用`setRequestHeader()`来添加HTTP头，然后在`send()`方法中规定希望发送的数据：

```
<html>
<head>
<script>
function sendXML() {
    var xmlhttp;
    if (window.XMLHttpRequest) {
        xmlhttp = new XMLHttpRequest();
    } else {
        xmlhttp = new ActiveXObject("Microsoft.XMLHTTP");
    }
    xmlhttp.onreadystatechange = function() {
        if (xmlhttp.readyState == 4 && xmlhttp.status == 200) {
            document.getElementById("myDiv").innerHTML = xmlhttp.responseText;
        }
    }
    xmlhttp.open("POST", "/ajax/demo_post2.asp", true);
    xmlhttp.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
    xmlhttp.send("fname=Bill&lname=Gates");
}

</script>
</head>

<body>
<h2>AJAX</h2>
<button onclick = "sendXML()">请求数据</button>
<div id = "myDiv"></div>
</body>
</html>
```

#### 3. 服务器响应
responseText：获得字符串形式的响应数据
responseXML：获得XML形式的响应数据，获得之后需要解析

```
<html>
<head>
<script>
function getXML() {
    var xmlhttp, txt, x;
    if (window.XMLHttpRequest) {
        xmlhttp = new XMLHttpRequest();
    } else {
        xmlhttp = new ActiveXObject("Microsoft.XMLHTTP");
    }
    xmlhttp.onreadystatechange = function() {
        if (xmlhttp.readyState == 4 && xmlhttp.status == 200) {
            xmlDoc = xmlhttp.responseXML;
            txt = "";
            x = xmlDoc.getElementsByTagName("title");
            for (var i = 0; i < x.length; i++) {
                txt = txt + x[i].childNodes[0].nodeValue + "<br/>";
            }
            document.getElementById("myDiv").innerHTML = txt;
        }
    }
    xmlhttp.open("GET", "/example/xmle/books.xml", true);
    xmlhttp.send();
}
</script>
</head>

<body>
<h2>My Book Collection</h2>
<div id = "myDiv"></div>
<button onclick = "getXML()">获得我的图书收藏列表</button>
</body>

</html>
```

#### 4. onreadystatechange事件
![](https://upload-images.jianshu.io/upload_images/8407639-0b46c1e302a354f5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果存在着多个AJAX文件需要解析，则需要进行解耦。

```
<html>
<head>
<script>
var xmlhttp;
function loadXML(url, cfunc) {
    if (window.XMLHttpRequest) {
        xmlhttp = new XMLHttpRequest();
    } else {
        xmlhttp = new ActiveXObject("Microsoft.XMLHTTP");
    }
    xmlhttp.onreadystatechange = cfunc;
    xmlhttp.open("GET", url, true);
    xmlhttp.send();
}

function myFunc() {
    loadXML("/ajax/test1.txt", function() {
        if (xmlhttp.readyState == 4 && xmlhttp.status == 200) {
            document.getElementById("myDiv").innerHTML = xmlhttp.responseText;
        }
    });
}
</script>
</head>

<body>
<div id = "myDiv"><h2> Let AJAX change this text</h2></div>
<button onclick = "myFunc()">通过 AJAX 改变内容</button>
</body>

</html>
```

#### 4. jQuery + AJAX
jQuery的load()方法是简单但强大的AJAX方法，它从服务器加载数据，并把返回的数据放入被选元素中。

```
$(selector).load(URL, data, callback);
```

data（可选）：规定与请求一同发送的查询字符串键值对集合
callback（可选）：load()方法完成后执行的函数。

```
<!DOCTYPE html>
<html>
<head>
<script src = "/jquery/jquery-1.11.1.min.js"></script>
<script>
$(document).ready(function() {
    $("button").click(function() {
        $("#test").load("/example/jquery/demo_test.txt");
    });
});
</script>
</head>

<body>
<h3 id = "test">点击按钮</h3>
<button>获得外部的内容</button>
</body>

</html>
```

load()里面也可以加入过滤的参数：

```
$("#test").load("/demo_test.txt #p1");    //选取这个文件里面的id为p1的段落
```

可选的callback有三个参数：
- responseTxt：包含调用成功时的结果内容
- statusTXT：包含调用的对象
- xhr：包含XMLHttpRequest对象

```
$("button").click(function() {
    $("div1").load("demo_test.txt", function(responseTxt, statusTxt, xhr) {
        if (statusTxt == "success") {
            alert("外部内容加载成功!");
        }
        if (statusTxt == "error") {
            alert("Error: " + xrh.status + ": " + xhr.statusText);
        }
    });
});
```

#### 4. GET 和 POST
GET：

```
$("button").click(function() {
    $.get("demp_test.asp", function(data, status) {
            alert("Data: " + data + "\nStatus: " + status);
    });
});
```

POST：

```
$("button").click(function() {
    $.post("demo_test_post.asp",
    {
        name: "Hoben",
        city: "Guangzhou"
    },
    function(data, status) {
        alert("Data: " + data + "\nStatus: " + status);
    });
}));
```

asp文件如下：

```
<%
dim fname, city
fname = Request.From("name")
city = Request.From("city")
Response.Write("Dear " & fname & ".")
Response.Write("Hope you live well in " & city & ".")
%>
```

### 五. noConflict()
考虑到其他框架也会用到\$符号进行简写，使用`noConflict()`方法可以避免

```
$.noConflict();
jQuery(document).ready(function() {
    jQuery("button").click(function() {
         jQuery("p").text("Hi");
    });
});
```
