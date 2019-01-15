---
layout: blog
demo: true
new: true
istop: true
ico: bg-ico-web
title:  统计页面demo
create: 原创
tags:
- css
- html
- ECharts
background: green
background-image: 
date:   2019-01-15 19:50:00
category: 案例
---

## 前言

重新学习前端知识时，制作了一个统计页面demo。由于很久没有更新内容，所以将这次案例用到的内容整理了一下。  

查看<a href="https://hangforfreedom.github.io/some-cases/geekschain/statistics/index.html">案例</a>

## 一、栅格系统

仿照 <a href="https://panjiachen.github.io/vue-element-admin/#/dashboard">**vue-element-admin**</a> 的样式，进行栅格系统分布。可以根据屏幕的大小自适应。  
  
查看<a href="https://hangforfreedom.github.io/some-cases/栅格系统案例/index.html">栅格系统案例</a>  

```
/******row 栅格系统******/
.panel-row:before,
.panel-row:after{
    content: "";
    display: table;
    clear: both;
}
[class *= 'col-']{
    float: left;
    min-height: 1px;
    padding: 12px;
}
.col-1{ width: 16.66%; }
.col-2{ width: 25%; }
.col-3{ width: 50%; }
.col-4{ width: 66.664%; }
.col-5{ width: 83.33%; }
.col-6{ width: 100%; }

@media all and (max-width: 1000px){
    .col-md-1{ width: 16.66%; }
    .col-md-2{ width: 33.33%; }
    .col-md-3{ width: 50%; }
    .col-md-4{ width: 66.664%; }
    .col-md-5{ width: 83.33%; }
    .col-md-6{ width: 100%; }
}
@media all and (max-width: 608px){
    .col-sm-1{ width: 16.66%; }
    .col-sm-2{ width: 33.33%; }
    .col-sm-3{ width: 50%; }
    .col-sm-4{ width: 66.664%; }
    .col-sm-5{ width: 83.33%; }
    .col-sm-6{ width: 100%; }
}
/***** row 栅格系统 *****/
```

* 区分max-width和min-width的使用
    * max-with:
        1. 当盒子宽度小于max时，所显示的宽度是盒子本身的宽度
        2. 当盒子宽度大于max时，所显示的宽度是max的宽度
    * min-width:
        1. 当盒子宽度小于min时，所显示的宽度是min的宽度
        2. 当盒子宽度大于min时，所显示的宽度是盒子本身的宽度

解释

* max-width的含义是：限制盒子达到最大宽度是多少。当盒子的宽度大于这个宽度时，max-width会起作用，限制盒子在这个宽度中。
* min-width的含义是：规定盒子最小宽度只能是多少。当盒子的宽度小于这个宽度时，min-width会起作用，扩大盒子的宽度至min-width宽度  

```
@media only screen and ( max-width: 980px ){}

当页面小于960px时候执行其中的语句
```
```
@media only screen and ( min-width: 980px ){}

当页面大于980px时候执行其中的语句
```
```
@media screen and (min-width:960px) and (max-width:1200px){}

当页面宽度大于960px小于1200px时候执行其中的语句
```

---------------------
## 二、重新复习如何引用iconfont

1. 阿里的iconfont网站中选择icon，下载源码。
2. 将压缩包中的iconfont.css文件引入页面中。css文件中可以更改icon的颜色、font-size大小等
3. ```<i class="iconfont 类名"></i>```

--------------------
## 三、ECharts

* 中国地图在英文版：```https://ecomfe.github.io/echarts-examples/public/index.html```
* 折线图在中文版：```https://www.echartsjs.com/examples/```

可以直接Download下来，然后在下载的页面上Copy  
  
或者引用官网外部css、js链接，然后Copy demo

## 四、ECharts响应式

1. 限制柱状图的宽度：```barMaxWidth：30//设置柱状最大的宽度```
2. 设置y轴的label标签显示  
3. 设置图表响应式(单个)```在配置项最后加上语句window.onresize = myChart.resize;```
4. 设置多个图表响应式```在配置项最后加上下面语句 window.addEventListener(“resize”, function () { myChart.resize(); });```