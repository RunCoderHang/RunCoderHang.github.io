---
title:  可以输入也可以下拉选择的select
tags:
- js
- JQuery
date:   2018-05-06 14:19:00
categories: Demo
---

我们知道，一般``select``下拉框是只能选择的，不能用来输入内容的。而有时我们会遇到下拉框中没有要选择的信息项或者下拉选项特别多时，我们可以让``select``变成``text``，允许用户输入想要的内容，同时还可以在输入的时候将包含关键字的项也列出来，供快速选择。

查看<a href="https://hangforfreedom.github.io/some-cases/demo-2/demo.html" target="_blank">案例</a>

本文将用实例和大家分享一款基于``jQuery``的下拉框插件，它允许用户输入内容，同时下拉选项中会及时匹配相关选项，支持键盘操作，还支持``html``选项内容，当然还能让下拉的过程带有动画效果。我们来看下如何使用。  


## 实现  

 * <b>HTML结构</b>

下面是一个基本的``select``下拉框

```html
<select id="editable-select">
    <option>JAVA</option>
    <option>C</option>
    <option>C++</option>
    <option>C#</option>
    <option>PYTHON</option>
    <option>HTML&CSS</option>
    <option>PHP</option>
</select>
```

此外还需要加载``jQuery``库和``jquery.editable-select.js``文件  

传送门，走你！<a href="https://hangforfreedom.github.io/some-cases/demo-2/js/jquery-3.2.1.min.js" download="">jQuery下载</a>  <a href="https://hangforfreedom.github.io/some-cases/demo-2/js/jquery-editable-select.min.js">jquery.editable-select.js下载</a>  

 * <b>jQuery</b>

只需要以下代码就能实现传统的下拉框变成有输入功能的下拉框了。

```js
$('#editable-select').editableSelect({
    effects: 'slide'
});
```

其实我们细看插件代码就会发现，作者是将原有的``select``处理了下，变成了一个输入表单``text``和一个列表``ul``。这样``text``可以输入，下拉选项则用``ul``面板，这样一来``ul``里的选项就可以添加任意``html``代码了，``demo``中有示例。然后通过使用``CSS``以及``js``技术可以实现下拉弹出、输入查找匹配功能。  

 * <b>选项设置</b>

<b>filter</b>：过滤，即当输入内容时下拉选项会匹配输入的字符，支持中文，true/false，默认true。  

<b>effects</b>：动画效果，当触发弹出下拉选择框时的下拉框展示过渡效果，有default，slide，fade三个值，默认是default。  

<b>duration</b>：下拉选项框展示的过渡动画速度，有fast，slow，以及数字（毫秒），默认是fast。  

 * <b>事件</b>

<b>onCreate</b>：当输入时触发。  

<b>onShow</b>：当下拉时触发。  

<b>onHide</b>：当下拉框隐藏时触发。  

<b>onSelect</b>：当下拉框中的选项被选中时触发。  

事件调用方法：  

```js
$('#editable-select').editableSelect({
    onSelect: function (element) {
        alert("Selected!");
    }
});
```

此外，还支持键盘方向键、回车键、Tab键以及Esc键操作

jQuery Editable Select项目官网地址：<a href="https://github.com/indrimuska/jquery-editable-select" target="_blank">https://github.com/indrimuska/jquery-editable-select</a>  

原文链接：https://www.helloweba.net/javascript/348.html