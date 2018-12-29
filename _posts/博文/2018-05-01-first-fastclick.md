---
layout: blog
article: true
new: true
ico: bg-ico-book
title:  初识fastclick.js插件使用
create: 原创
tags:
- js
background: green
background-image: https://hangforfreedom.github.io/thumbnails/fastclick.png
date:   2018-05-01 21:41:00
category: 博文
---

<img src="https://hangforfreedom.github.io/thumbnails/fastclick.png" alt="fastclick">

---
## 初识``fastclick.js``  

为了<a href="https://hangforfreedom.github.io/2018/05/01/use-headroom.html">实现导航栏在滚动页面时消失</a>，我浏览一位博主的博客作为案例。在源码中，我发现一个``fastclick.js``插件，于是好奇地查找它的作用。  

>在移动页面开发上，会出现一个问题，click事件会有300ms的延迟，这让用户感觉像是网页卡顿了一样。实际上，这是浏览器为了更好的判断用户的双击行为，移动端浏览器都支持双击缩放或双击滚动的操作，比如一个链接，当用户第一次点击后，浏览器不能立刻判断用户确实要打开这个链接，还是想要进行双击的操作，因此几乎现在所有浏览器都效仿Safari当年的约定，在点击事件上加了300毫秒的延迟。  

因为这300ms的延迟，催生了``fastclick``的诞生。  


### ``fastclick``的兼容性  

``FastClick``能够兼容以下浏览器：

 * Mobile Safari on iOS 3 and upwards 
 * Chrome on iOS 5 and upwards 
 * Chrome on Android (ICS) 
 * Opera Mobile 11.5 and upwards 
 * Android Browser since Android 2 
 * PlayBook OS 1 and upwards  

---
## ``fastclick``什么时候不使用  

fastclick不附加任何监听器在桌面浏览器上面，所以如果你的项目不是针对的移动浏览器，那么就不要使用这个插件。  

Android 设备上的 google浏览器 （Chrome） 32+ 版本，在meta头信息中设置 width=device-width 没有300毫秒的延时，所以也无需使用本插件。  

```
<meta name="viewport" content="width=device-width, initial-scale=1">
```

>Chrome浏览器在安卓设备上的时候，设置meta头信息中 user-scalable=no 但是这样就无法让用户多点触控缩放网页了。  

>对于IE11 + 你可以设置``touch-action: manipulation;``来禁用通过双击放大某些元素例如：链接和按钮的，对于IE10使用``-ms-touch-action: manipulation``。  

---
## ``fastclick.js``的使用  

提供``fastclick.js``的源码地址：<a href="http://www.bootcdn.cn/fastclick/" target="_blank">http://www.bootcdn.cn/fastclick/</a>  
首先在HTML页面中添加外联``javascript``文件  

```
<script type="text/javascript" src="https://cdn.bootcss.com/fastclick/1.0.6/fastclick.min.js"></script>
```

还有内部``js``文件  

```
<script>
	if ('addEventListener' in document) {  
	    document.addEventListener('DOMContentLoaded', function() {  
	        FastClick.attach(document.body);  
	    }, false);  
	}
</script>
```

当然，如果使用``JQuery``，``js``可以写成如下  

```
<script>
	$(function() {  
	    FastClick.attach(document.body);  
	});
</script>
```

如果使用``Browserify``或者其他``CommonJS-style``系统，当你调用``require('fastclick')``时，``FastClick.attach``事件会被返回，加载``FastClick``最简单的方式就是下面的方法了：  

```
<script>
	var attachFastClick = require('fastclick');  
	attachFastClick(document.body);
</script>
```

完整案例如下：  

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>fastclick案例</title>
</head>
<body>
    <button id="click">click me!</button>
    <script type="text/javascript" src="https://cdn.bootcss.com/fastclick/1.0.6/fastclick.min.js"></script>
    <script type="text/javascript">
        if ('addEventListener' in document) {
            document.addEventListener('DOMContentLoaded', function(){
                FastClick.attach(document.body);
            }, false);
        }
        document.querySelector("#click").addEventListener("click",function(){
            alert("click me!");
        },false)
    </script>
</body>
</html>
```
