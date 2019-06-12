---
title:  js改变style样式和css样式
tags:
- js
- css
date: 2018-05-12 21:28:00
categories: JavaScript
---

在很多情况下，都需要对网页上元素的样式进行动态的修改。在JavaScript中提供几种方式动态的修改样式，下面将介绍方法的使用、效果、以及缺陷。  

 - 使用div.style.CSS来修改样式表的语法
 - 使用div.style.cssText来修改嵌入式的css
 - 使用div.className来修改样式表的类名
 - 使用更改外联的css文件，从而改变元素的css

日常贴案例，点击<a href="https://hangforfreedom.github.io/some-cases/demo-8/demo.html" target="_blank">查看</a>  

## 详述

下面是一段``html``代码和``css``代码用来解释上面方法的区别。

首先是基本的布局代码：

```css
.main{
    display: flex;
    width: 600px;
    height: 300px;
    margin: 100px auto;
}
.style1{
    width: 130px;
    height: 130px;
    background-color: #337ab7;
    color: #fff;
    font-size: 14px;
    font-weight: bold;
    text-align: center;
    line-height: 45px;
    margin: 85px auto;
}
.tool{
    width: 242px;
    height: 300px;
    padding: 10px 0;
}
.btn{
    width: 242px;
    margin: 17px auto;
}
```

```html
<div class="main">
    <div id="tool" class="tool">
        <input  class="btn btn-info" type="button" value="【div.style.CSS】更改样式" onclick="changeBackgroundColor()">
        <input class="btn btn-info" type="button" value="【div.style.cssText】更改样式" onclick="changeFontSize()" >
        <input class="btn btn-info" type="button" value="【div.className】更改样式" onclick="addRadius()" >
        <input class="btn btn-info" type="button" value="更改外联css样式" onclick="recover()" >
    </div>
    <div id="div" class="style1">
        我不是按钮<br>它才是<br><<
    </div>
</div>
```

#### 修改样式表的语法

使用修改样式表的语法来修改样式，话说怎么这么绕口呢···贴代码

```js
function changeBackgroundColor(){
    var div = document.getElementById("div");
    div.style.backgroundColor="#06d6a0";
}
```

该段代码修改``div``的背景颜色，在浏览器中打开调试工具，可以发现``div``标签中多了一个``style="background-color: #06d6a0;``的样式。所以它是直接在元素中添加样式的修饰语法来更改样式的。  

#### 修改嵌入式的css

```js
function changeFontSize(){
    var div = document.getElementById("div");
    div.style.cssText = "font-size: 7px;"
}
```

该代码是修改``div``中字体的大小。同样，在浏览器调试工具中，该功能和第一个一样。  

**缺点**：它很霸道，因为它在标签上添加样式后就不会因为其他功能而消失。也就是说只能通过刷新页面的方式来重置标签的样式。

#### 修改样式表的类名

```js
function addRadius(){
    var div = document.getElementById("div");
    //div.className = "style2";
    div.setAttribute("class", "style2");
}
```

该功能是将``div``变成圆形。此方法可以写多个类，并对不同的类名添加样式，然后使用此代码进行修改类名实现元素样式的修改。  

用这种方式来修改css比上面的效果要好很多

#### 更改外联的``css``文件

```js
<link rel="stylesheet" href="css1.css" id="css">

function recover(){
    var div = document.getElementById("css");
    div.setAttribute("href","css2.css");
}
```

该功能是将``div``恢复到原来的样式中。前面也说到，第二种方法太过霸道，所以字体并不能恢复过来。  

此方法是写两个外联的``css``文件，如``css1.css``和``css2.css``。通过``id``找到``css1.css``，然后修改其链接为``css2.css``。所显示的样式都是``css2.css``文件中的样式。

#### ``JS``控制``CSS``样式表的语法对照表

|           <b>盒子标签和属性对照</b>           |                               |
|-----------------------------------------------|-------------------------------|
| CSS语法(不区分大小写)                         | JavaScript语法(区分大小写)    |
| border                                        | border                        |
| border-bottom                                 | borderBottom                  |
| border-bottom-color                           | borderBottomColor             |
| border-bottom-style                           | borderBottomStyle             |
| border-bottom-width                           | borderBottomWidth             |
| border-color                                  | borderColor                   |
| border-left                                   | borderLeft                    |
| border-left-color                             | borderLeftColor               |
| border-left-style                             | borderLeftStyle               |
| border-left-width                             | borderLeftWidth               |
| border-right                                  | borderRight                   |
| border-right-color                            | borderRightColor              |
| border-right-style                            | borderRightStyle              |
| border-right-width                            | borderRightWidth              |
| border-style                                  | borderStyle                   |
| border-top                                    | borderTop                     |
| border-top-color                              | borderTopColor                |
| border-top-style                              | borderTopStyle                |
| border-top-width                              | borderTopWidth                |
| border-width                                  | borderWidth                   |
| clear                                         | clear                         |
| float                                         | floatStyle                    |
| margin                                        | margin                        |
| margin-bottom                                 | marginBottom                  |
| margin-left                                   | marginLeft                    |
| margin-right                                  | marginRight                   |
| margin-top                                    | marginTop                     |
| padding                                       | padding                       |
| padding-bottom                                | paddingBottom                 |
| padding-left                                  | paddingLeft                   |
| padding-right                                 | paddingRight                  |
| padding-top                                   | paddingTop                    |
|                                               |                               |
| <b>颜色和背景标签和属性对照</b>               |                               |
| CSS 语法（不区分大小写）                      | JavaScript 语法（区分大小写） |
| background                                    | background                    |
| background-attachment                         | backgroundAttachment          |
| background-color                              | backgroundColor               |
| background-image                              | backgroundImage               |
| background-position                           | backgroundPosition            |
| background-repeat                             | backgroundRepeat              |
| color                                         | color                         |
|                                               |                               |
| <b>样式标签和属性对照</b>                     |                               |
| CSS语法（不区分大小写）                       | JavaScript 语法（区分大小写） |
| display                                       | display                       |
| list-style-type                               | listStyleType                 |
| list-style-image                              | listStyleImage                |
| list-style-position                           | listStylePosition             |
| list-style                                    | listStyle                     |
| white-space                                   | whiteSpace                    |
|                                               |                               |
| <b>文字样式标签和属性对照</b>                 |                               |
| CSS 语法（不区分大小写）                      | JavaScript 语法（区分大小写） |
| font                                          | font                          |
| font-family                                   | fontFamily                    |
| font-size                                     | fontSize                      |
| font-style                                    | fontStyle                     |
| font-variant                                  | fontVariant                   |
| font-weight                                   | fontWeight                    |
|                                               |                               |
| <b>文本标签和属性对照</b>                     |                               |
| CSS 语法（不区分大小写）                      | JavaScript 语法（区分大小写） |
| letter-spacing                                | letterSpacing                 |
| line-break                                    | lineBreak                     |
| line-height                                   | lineHeight                    |
| text-align                                    | textAlign                     |
| text-decoration                               | textDecoration                |
| text-indent                                   | textIndent                    |
| text-justify                                  | textJustify                   |
| text-transform                                | textTransform                 |
| vertical-align                                | verticalAlign                 |
|                                               |                               |
| <b>规律:"-"去掉,并把-后面的首字母换成大写</b> |                               |