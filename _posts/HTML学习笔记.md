###一.HTML简介
####1.什么是HTML
HTML是用来描述网页的一种语言。
- HTML指的是超文本标记语言
- HTML不是一种编程语言，而是一种标记语言
- 标记语言是一套标记标签
- HTML使用标记标签来描述网页

####2.HTML标签
- HTML标签是由尖括号包围的关键词，比如<html>
- HTML标签通常是成对出现的，比如<b>和</b>
- 标签对中的第一个标签是`开始标签`，第二个标签是`结束标签`
- 开始和结束标签也被称为开放标签和闭合标签

####3.HTML文档=网页
Web浏览器的作用是读取HTML文档，并以网页的形式去显示出它们。浏览器不会显示HTML标签，而是使用标签来解释页面的内容。
```
<html>
<body>

<h1>我的第一个标题</h1>

<p>我的第一个段落。</p>

</body>
</html>
```
- <html>和</html>之间的文本描述网页
- <body>和</body>之间的文本是可见的页面内容
- <h1>和</h1>之间的文本被显示为标题
- <p>和</p>之间的文本被显示为段落

###二.简单的HTML标签
####1.四大元素
- HTML 标题（Heading）是通过 <h1> - <h6> 等标签进行定义的。
- HTML 段落是通过 <p> 标签进行定义的。
- HTML 链接是通过 <a> 标签进行定义的。
```
<a href="http://www.baidu.com"> This is a link </a>
```
- HTML 图像是通过 <img> 标签进行定义的。
```
<img src = "wcschool.jpg" width = "104" height = "141"/>
```
####2.空的HTML元素
没有内容的HTML元素被称为空元素，空元素是在开始标签中关闭的。用<br />表示空行。

###三.HTML属性
属性总是以名称/值对的形式出现，如：name = "value"，且总是在HTML元素的开始标签中规定。

下面的代码中，href属性指定链接的地址，<a>标签定义链接。
```
<a href="http://www.baidu.com"> This is a link </a>
```

下面的代码中，align属性指定对齐方式
```
<h1 align = "center"> message </h1>
```

下面的代码中，bgcolor属性指定背景颜色的附加信息
```
<body bgcolor = "yellow"> body </body>
```

属性值始终被引号包围，如果内容有双引号，则需要用单引号来描述，如
```
name='Bill "HelloWorld" Gates'
```

###四.HTML样式
- style属性：用于改变HTML元素的样式，它提供了一种改变所有HTML元素的样式的通用方法，可以使用style属性直接将样式添加到HTML元素，或者间接地在独立的样式表中（CSS文件）进行定义。

下面的代码中，background-color 属性为元素定义了背景颜色：
```
<html>

<body style = "background-color:yellow">
<h2 style = "background-color:red"> heading2 </h2>
<p style = "background-color:green"> paragraph </p>
</body>

</html>
```

下面的代码中，font-family、color 以及 font-size 属性分别定义元素中文本的字体系列、颜色和字体尺寸：
```
<html>

<body>
<h2 style = "font-family:verdana"> heading2 </h2>
<p style = "font-family:arial; color:red; font-size:20px"> paragraph </p>
</body>

</html>
```

下面的代码中，text-align 属性规定了元素中文本的水平对齐方式：
```
<html>

<body>
<h1 style = "text-align: center"> heading1 </h1>
<p> paragraph </p>
</body>

</html>
```

使用样式的三种方法：
- 外部样式表：当样式需要被应用到**很多页面**的时候，外部样式表是理想的选择。使用外部样式表，可以通过更改一个文件来改变整个站点的外观。
```
<head>
<link rel = "stylesheet" type = "text/css" href = "mystyle.css">
</head>
```

- 内部样式表：当单个文件需要特别样式的时候，就可以使用内部样式表，可以在head部分通过<style>标签定义内部样式表。
```
<head>

<style type = "text/css">
body {background-color: red}
p {margin-left: 20px}
</style>

</head>
```
- 内联样式：当特殊的样式需要应用到个别元素时，就可以使用内联样式。使用内联样式的方法是在相关的标签中使用样式属性。样式属性可以包含任何CSS属性。
```
<p style = "color:red; margin-left: 20px">
This is a paragraph
</p>
```

###五.分组标签
- 块元素：在浏览器显示时，通常会以新行来开始（和结束），如<h1>, <p>, <ul>, <table>
- 内联元素：在显示时不会以新行开始，如<b>, <td>, <a>, <img>
- <div>元素：是块级元素，可用于组合其他HTML元素的容器，它没有特定的含义，浏览器会在其前后显示折行。
```
<body>

<h1>NEWS WEBSITE</h1>
<p>some text.</p>
...

<div class = "news">
<h2>Headline1</h2>
<p>some text...</p>
</div>

<div class = "news">
<h2>Headline2</h2>
<p>some text...</p>
</div>
...

</body>
```
在上面的HTML代码模拟了新闻网站的结构，每个div把每条新闻的标题和摘要组合在一起，div为文档添加了额外的结构。

由于这些div属于同一类元素，所以可以使用class="news"对这些div进行标识，这么做不仅为div添加了合适的语义，而且便于进一步使用样式对div进行格式化，一举两得。

- <span>元素：是内联元素，可以作为文本的容器，也没有特定的含义。用class或id属性来标识<span>元素，这样就可以对<span>标识的元素进行处理。
```
在HTML中：
<p class = "tip"> <span>提示：</span>... ... ...</p>

在CSS中：
p.tip span {
    font-weight: bold;
    color:#ff9955;
}
```

###六.HTML类
对HTML进行分类，使我们能够为元素的类定义CSS样式，为相同的类设置相同的样式，或者为不同的类设置不同的样式。

<div>使用类：
```
<!DOCTYPE html>
<html>
<head>
<style>
.cities {
    background-color:black;
    color:white;
    margin:20px;
    padding:10px;
}
</style>
</head>

<body>

<div class = "cities">
<h2>London</h2>
<p>About London</p>
<p>Description</p>
</div>

<div class = "cities">
<h2>Paris</h2>
<p>About Paris</p>
<p>Description</p>
</div>

</body>
</html>
```
![](https://upload-images.jianshu.io/upload_images/8407639-76239496fa71b4b8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<span>使用类：
```
<!DOCTYPE html>
<html>
<head>
<style>
span.red {
    color:red;
}
</style>
</head>

<body>
<h1>我的<span class = "red">重要</span>标题</h1>
</body>

</html>
```
![](https://upload-images.jianshu.io/upload_images/8407639-d5435fb3ad1f915b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###七.HTML布局
用CSS控制位置、颜色等布局，其他标签使用CSS填写内容：
```
<!DOCTYPE html>
<html>
<head>
<style>
#header {
    background-color:black;
    color:white;
    text-align:center;
    padding:5px;
}
#nav {
    line-height:30px;
    background-color:#eeeeee;
    height:300px;
    width:100px;
    float:left;
    padding:5px;
}
#section {
    width:350px;
    float:left;
    padding:10px;
}
#footer {
    background-color:black;
    color:white;
    clear:both;
    text-align:center;
    padding:5px;
}
</style>
</head>

<body>

<div id = "header">
<h2>
City Gallery
</h2>
</div>

<div id = "nav">
London<br>
Paris<br>
Tokyo<br>
</div>

<div id = "section">
<h1>London</h1>
<p> London is the capital city of England. It is the most populous city in the United Kingdom, with a metropolitan area of over 13 million inhabitants.</p>
<p> Standing on the River Thames, London has been a major settlement for two millennia, its history going back to its founding by the Romans, who named it Londinium.</p>
</div>

<div id = "footer">
<p> Copyright ? W3Schools.com</p>
</div>

</body>

</html>
```
![](https://upload-images.jianshu.io/upload_images/8407639-96210930823b8fcc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###八.表格
- 每个表格由<table>开始
- 每个表格行由<tr>开始
- 每个表格数据由<td>开始
- 表格的表头由<th>定义，显示为粗体居中的文本
- 空单元格 <td> \&nbsp; </td>
- 标题<caption>
- 跨行：<th rowspan="2"></th>
- 控制padding：<table cellpadding = "10">
- 控制spacing：<table cellspacing = "10">
- 对齐：<td align = "left>
```
<h4>两行三列：</h4>
<table border="1">
<tr>
<td>100</td>
<td>200</td>
<td>300</td>
</tr>

<tr>
<td>400</td>
<td>500</td>
<td>600</td>

</table>
```
![](https://upload-images.jianshu.io/upload_images/8407639-027fa9b5f9348d36.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###九.列表
- 无序列表：
```
<!--- 可以通过改变<ul type = "disc/square/circle">来改变显示 --->
<ul>
<li>a</li>
<li>b</li>
</ul>
```
- 有序列表：
```
<!--- 可以通过改变<ol type = "A/a/I/i">来改变显示 --->
<ol>
<li>a</li>
<li>b</li>
</ol>
```
###十.响应式设计
- 以可变尺寸传递网页
- 对于平板和移动设备是必需的
- 可以使用现成的CSS框架（如Bootstrap）

###十一.HTML脚本
<script>定义客户端脚本，如JavaScript，既可以包含脚本语句，也可以通过src属性指向外部脚本文件。

<noscript>用于显示禁用脚本的提示
```
<script type="text/javascript">
document.write("Hello World!")
</script>
<noscript>Your browser does not support JavaScript!</noscript>
```

###十二.头部元素
- <head>：是所有头部元素的容器，可以包含脚本，指示浏览器在何处可以找到样式表，提供元信息等：<title>、<base>、<link>、<meta>、<script>、<style>

- <title>：定义文档的标题，如浏览器工具栏中的标题、页面被添加到收藏夹时显示的标题、在搜索引擎结果中的页面标题。

- <base>：为页面上的所有链接规定默认地址或者默认目标

- <link>：定义文档与外部资源之间的关系，最常用于连接样式表：
```
<head>
<link rel = "stylesheet" type = "text/css" href = "mystyle.css" />
</head>
```

- <style>：为HTML文档定义样式信息，可以在style元素内规定HTML元素在浏览器中呈现的样式。

- <meta>：是关于数据的信息，提供关于HTML文档的元数据（不会显示在页面上，但对于机器是可读的），典型情况下，用于规定页面的描述、关键词、文档的作者、最后修改时间以及其他元数据。

- <script>：用于定义客户端脚本，如JavaScript

###十三.HTML统一资源定位器
URL：用于定位万维网上面的文档
>scheme://host.domain:port/path/filename
- scheme：定义因特网服务的类型，最常见的类型是http
- host：定义域主机（http的默认主机是www）
- domain：定义因特网域名，如baidu.com
- :port ：定义主机上的端口号，http默认端口是80
- path：定义服务器的路径（如果省略，则文档必须位于网站的根目录中）
- filename：定义文档/资源的名称
![](https://upload-images.jianshu.io/upload_images/8407639-d127c77e65fddeab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###十四.表单
- <form>元素：HTML表单用于收集用户输入。包含不同类型的input元素、复选框、单选按钮、提交按钮等待。

- <input>：最重要的表单元素，有很多形态，根据不同的type属性。如text(定义常规文本输入，默认宽度20字符)，radio（定义单选按钮），submit（定义提交按钮）
```
<html>
<body>

<form>
First name: <br/>
<input type = "text" name = "firstname"/>
</br>
Last name: <br/>
<input type = "text" name = "lastname"/>
</form>

</body>
</html>
```

```
<html>
<body>

<form>
<input type = "radio" name = "sex" value = "male" checked> Male
<br/>
<input type = "radio" name = "sex" value = "female"> Female
</form>

</body>
</html>
```

```
<html>
<body>

<form action = "/demo/demo_form.asp">
First name: </br>
<input type = "text" name = "firstname" value = "Mickey"/>
<br/>
Last name: </br>
<input type = "text" name = "lastname" value  = "Mouse"/>
<br/><br/>
<input type = "submit" value = "Submit"/>
</form>

<!--- 如果您点击提交，表单数据会被发送到名为 demo_form.asp 的页面。--->

</body>
</html>
```

- `Action属性`定义在提交表单时执行的动作，通常，表单会被提交到web服务器上的网页。
```
<form action = "action_page.php">
```

- `method属性`规定在提交表单时所用的HTTP方法（GET或者POST）
```
<form action = "action_page.php" method = "GET">
```

- `name属性`：每个输入字段必须设置一个name属性。

- 组合表单：<fieldset>元素组合表单中的相关数据，<legend>元素为<fieldset>元素定义标题。
```
<!DOCTYPE html>
<html>
<body>

<form action="/demo/demo_form.asp">
<fieldset>
<legend>Personal information:</legend>
First name:<br>
<input type="text" name="firstname" value="Mickey">
<br>
Last name:<br>
<input type="text" name="lastname" value="Mouse">
<br><br>
<input type="submit" value="Submit">
</fieldset>
</form>

</body>
</html>
```
加了<fieldset>之后，整个表单组合成了一个整体。

![](https://upload-images.jianshu.io/upload_images/8407639-173035ad0ca112d7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- <select>：下拉列表
```
<!DOCTYPE html>
<html>
<body>

<form action = "/demo/demo_form.asp">
<select name = "cars">
<option value = "volvo"> Volvo </option>
<option value = "saab"> Saab </option>
<option value = "audi"> Audi </option>
</select>
<br/><br/>
<input type = "submit">
</form>

</body>
</html>
```

- <button>：可点击的按钮
```
<button type = "button" onclick = "alert('Hello World!')"> Click Me! </button>
```