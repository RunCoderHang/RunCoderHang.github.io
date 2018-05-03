---
layout: blog
demo: true
istop: true
new: true
ico: bg-ico-code
title:  headroom.js实现导航栏在页面滚动时消失
create: 原创
tags:
- js
- css
- animation
- headroom
background: orange
background-image: 
date:   2018-05-01 18:11:00
category: 案例
---

## 前言

浏览一位博主的博客时，发现与我的博客主题很相似。但是，我的博客中头部``header``不能在页面滚动时消失。于是，我开始添加效果，并做出了相应的<a href="https://hangforfreedom.github.io/some-cases/demo-5/demo.html" target="_blank">案例</a>提供参考。  
<strong>Ps：</strong> 案例是基于``bootstrap``的响应式设计

---
## 分析

实现这个功能其实并不难。主要就是：  

- ``header``需要的``css``样式，如：``position: fixed;``
- 给定``header``实现动画的``css``样式，如``animation``
- ``CSS3``写入``@keyframes``添加动画
- ``js``的类名转换作用

下面我们开始列出代码，实现功能。

---
## 实现

给``header``一个``id="header"``这个在后面的``js``代码中会用到。  


### ``header``的``css``

```css
#header{
    position:fixed;
    z-index:10;
    top:0;
    width:100%;
    height: 80px;
    background-color: #f5f5f5;
    box-shadow: 0px 3px 10px #888;
}
```

其中``position: fixed``和``top: 0``可以让``header``相对``body``固定在顶部，而``z-index: 10``，则是让``header``处于页面的最前面，滑动页面时，``header``则一直覆盖在其它元素上面。  


### 给定``header``实现动画的``css``

```css
.header.animated{
    -webkit-animation-duration:.5s;
    animation-duration:.5s;
    -webkit-animation-fill-mode:both;
    animation-fill-mode:both;
}

.header.animated.slideUp{
    -webkit-animation-name:slideUp;
    animation-name:slideUp;
}

.header.animated.slideDown{
    -webkit-animation-name:slideDown;
    animation-name:slideDown;
}
```

``animated``、``slideDown``、``slideUp`` 分别是``header``的类名，只不过在html中不需要给``header``添加这些类名，因为``js``会添加类名，显示动画效果。

``slideDown``指鼠标上滚时，目标元素向下滑动，意为出现  

``slideUp``指鼠标下滚时，目标元素向上滑动，意为消失  

|         名称        |            作用            |
|---------------------|----------------------------|
| -webkit-            | 适用于Chrome 和 Safari     |
| animation-duration  | 动画执行一次持续的时长     |
| animation-fill-mode | 规定对象动画时间之外的状态 |
| animation-name      | 规定 @keyframes 动画的名称 |  


### ``@keyframes``添加动画

 ```css
@-webkit-keyframes unpinned{
    0%{-webkit-transform:translateY(0)}
    100%{-webkit-transform:translateY(50px)}
}

@keyframes unpinned{
    0%{-webkit-transform:translateY(0);transform:translateY(0)}
    100%{-webkit-transform:translateY(50px);transform:translateY(50px)}
}

@-webkit-keyframes pinned{
    0%{-webkit-transform:translateY(50px)}
    100%{-webkit-transform:translateY(0)}
}

@keyframes pinned{
    0%{-webkit-transform:translateY(50px);transform:translateY(50px)}
    100%{-webkit-transform:translateY(0);transform:translateY(0)}
}

@-webkit-keyframes slideDown{
    0%{-webkit-transform:translateY(-80px)}
    100%{-webkit-transform:translateY(0)}
}

@keyframes slideDown{
    0%{-webkit-transform:translateY(-80px);transform:translateY(-80px)}
    100%{-webkit-transform:translateY(0);transform:translateY(0)}
}

@-webkit-keyframes slideUp{
    0%{-webkit-transform:translateY(0)}
    100%{-webkit-transform:translateY(-80px)}
}

@keyframes slideUp{
    0%{-webkit-transform:translateY(0);transform:translateY(0)}
    100%{-webkit-transform:translateY(-80px);transform:translateY(-80px)}
}
 ```

|        名称        |                                         作用                                         |
|--------------------|--------------------------------------------------------------------------------------|
| @keyframes         | 规定动画(Internet Explorer 10、Firefox 以及 Opera 支持)                              |
| @-webkit-keyframes | Chrome 和 Safari 支持                                                                |
| transform          | 可以对网页元素进行变换的属性，比如旋转，缩放，移动                                   |
| translate          | translate是transform中的属性，translate指的是元素的移动，括号中的值分别指y轴的平移量 |  


### ``js``的类名转换作用

提供使用``headroom.js``源码的地址<a href="http://www.bootcdn.cn/headroom/" target="_blank">http://www.bootcdn.cn/headroom/</a>  

 - 引用``js``文件

```html
<script src="https://cdn.bootcss.com/headroom/0.9.4/headroom.min.js"></script>
```

 - 写外部``js``代码

```js
var header = new Headroom(document.getElementById("header"),
{
    // scroll tolerance in px before state changes
    tolerance: 5,
    // 在元素没有固定之前，垂直方向的偏移量（以px为单位）
    offset: 0,
    // 对于每个状态都可以自定义css classes 
    classes: {
        // 当元素初始化后所设置的class
        initial: "animated",
        // 向上滚动时设置的class
        pinned: "slideDown",
        // 向下滚动时所设置的class
        unpinned: "slideUp"
      }
});
header.init();
```

以上一些参数的解释有些抽象，此网址的参数改变更为具体<a href="http://www.bootcss.com/p/headroom.js/playroom/" target="_blank">http://www.bootcss.com/p/headroom.js/playroom/</a>

<a href="https://hangforfreedom.github.io/some-cases/demo-5/demo.html" target="_blank">我的案例 https://hangforfreedom.github.io/some-cases/demo-5/demo.html</a>