---
layout: blog
demo: true
istop: true
new: true
ico: bg-ico-code
title:  七种实现右侧固定，左侧自适应两栏布局的方法
create: 转载
tags:
- html
- css
- flex
- grid
background: blue
background-image: https://hangforfreedom.github.io/style/images/lightblue.png
date:   2018-04-30 21:20:00
category: 案例
---
<!-- 
* content
{:toc}

 -->
参考<a href="https://segmentfault.com/a/1190000010698609" target="_blank">文章</a>实现了``右边固定，左边自适应``的两栏布局的七种方法。最终的效果，可以查看<a href="https://hangforfreedom.github.io/some-cases/demo-4/demo.html" target="_blank">这里</a>。  

与原案例不同的是右边``div``和左边``div``在html源码中的书写的先后顺序也是不同的。  

所以我将右边div分为两个类，一个是书写在左边div上面的``top-right``，一个是书写在左边div后面的``bottom-right``。代码如下：  

```
<div class="wrapper wrapper-inline-block" id="wrapper">
    <div class="top-right">     /*书写在左侧div之上*/
        我是右边div：top-right<br><br>
        基本样式：两个div相距20px，右侧div宽200px<br><br><br><br><br>
    </div>
    <div class="left">
        我是左边div：left<br>
        高度有可能会很小，也可能很大。<br>
        我是自适应。我是自适应。我是自适应。我是自适应。我是自适应。我是自适应。我是自适应。
        <br>
    </div>
    <div class="bottom-right">  /*书写在左侧div之下*/
        我是右边div：bottom-right<br><br>
        基本样式：两个div相距20px，右侧div宽200px<br><br><br><br><br>
    </div>
</div>
```

基本的样式是：两个盒子相距``20px``，右盒子宽``200px``，左盒子自适应。基本的CSS样式如下：  

```
.wrapper{
    padding: 15px 20px;
    border: 1px solid #f60;
}

.left{
    margin-right: 20px;
    border: 5px solid #ddd;
}

.top-right,
.bottom-right{
    width: 200px;
    border: 5px solid #ddd;
}
```

下面的代码就是基于这套基本代码做覆盖，通过容器添加不同的类来实现效果。  


## 双``inline-block``方案
<hr style="border: 0.5px solid #ccc;">

```
.wrapper-inline-block{
    box-sizing: content-box;
    font-size: 0;   /*消除空格的影响*/
}

.wrapper-inline-block .left,
.wrapper-inline-block .bottom-right{
    display: inline-block;
    vertical-align: top;    /*顶端对齐*/
    font-size: 14px;
    box-sizing: border-box;
}

.wrapper-inline-block .left{
    width: calc(100% - 225px);
}
```

这种方法是通过``width: calc(100% - 225px)``来动态计算左盒子的宽度。``225px``指的是左盒子距离右盒子的距离，以及右盒子具体的宽度``(content+padding+border)``，以及计算父容器宽度的``100%``需要减去的数值。同时，还需要知道左侧盒子的宽度是否包含``border``的宽度。  
在这里，为了简单的计算左侧盒子准确的宽度，设置子元素的``box-sizing: border-box;``以及父元素的``box-sizing: content-box;``。  
同时，作为两个``inline-block``的盒子，必须设置``vertical-align``来使其顶端对齐。  
另外，为了准确的应用计算出来的宽度，需要消除``div``之间的空格，需要通过设置父容器的``font-size: 0;``，或者注销消除``html``中的空格等方法。  
<strong>缺点：</strong>  
 * 需要知道右侧盒子的宽度，两个盒子的距离，还要设置各个元素的``box-sizing``  
 * 需要消除空格字符的影响  
 * 需要设置``vertical-align: top;``满足顶端对齐。  


## 双``float``方案  
<hr style="border: 0.5px solid #ccc;">

```
.wrapper-double-float{
    overflow: auto;    /*清除浮动*/
    box-sizing: content-box;
}

.wrapper-double-float .left,
.wrapper-double-float .bottom-right{
    float: left;
    box-sizing: border-box;
}

.wrapper-double-float .left{
    width: calc(100% - 225px);
}
```

本方案和双``inline-block``方案原理相同，都是通过动态计算宽度来实现自适应。但是，由于浮动的``block``元素在有空间的情况下会依次紧贴，排列在一行，所以无需设置``display: inline-block;``，自然也就减少了顶端对齐，空格字符占空间等问题。  

> A floated box is shifted to the left or right until its outer edge touches the containing block edge or the outer edge of another float.  

不过由于应用了浮动，父元素需要清除浮动。  
<strong>缺点：</strong>  
 * 需要知道右侧盒子的宽度，两个盒子的距离，还要设置各个元素的``box-sizing``
 * 父元素需要清楚浮动  


## ``float+margin-right``方案  
<hr style="border: 0.5px solid #ccc;">

```
.wrapper-float{
    overflow: hidden;   /*清除浮动*/
}

.wrapper-float .left{
    margin-right: 225px
}

.wrapper-float .top-right{
    float: right;
}
```

上面两种方案都是利用了CSS的``calc()``函数来计算宽度值。下面两种方案则是利用了``block``级别的元素盒子的宽度具有<strong>填满父容器，并随着父容器的宽度自适应的流动特性</strong>。  
但是``block``级别的元素都是独占一行的，所以要想办法让两个``block``排列到一起。  

> 我们知道，``block``级别的元素会认为浮动的元素不存在，但是``inline``级别的元素能识别到浮动的元素。这样，``block``级别的元素就可以和浮动的元素同处一行了。  

以上是原作者在文章中解释到的。不过，在我的案例中，我尝试着改动右侧``div``和左侧``div``的书写顺序，才实现了右侧固定，左侧自适应的效果。所以代码中的类为``top-right``。  
为了让右侧盒子和左侧盒子保持距离，需要为右侧盒子留出足够的距离。这个距离的大小为右侧盒子的宽度以及两个盒子之间的距离之和。然后将改值设置为左侧盒子的``margin-right``。  

<strong>缺点：</strong>
 * 需要清除浮动
 * 需要计算左侧盒子的``margin-right``  


## 使用``absolute+margin-right``方案  
<hr style="border: 0.5px solid #ccc;">

另外一种让两个``block``排列到一起的方法是对右盒子使用``position: absolute;``的绝对定位。这样，左侧盒子也能无视掉它。  

```
.wrapper-absolute{
    position: relative;
}

.wrapper-absolute .top-right{
    position: absolute;
    right: 20px;
}

.wrapper-absolute .left{
    margin-right: 225px;
}
```

当然，右侧盒子使用绝对定位后，不仅要调整它的位置``right: 20;``，还要对其父元素使用相对定位``position: relative``。这样保证了和左侧盒子距离的准确性。  
<strong>缺点：</strong>
 * 使用了绝对定位，则需要在父元素中使用相对定位``position: relative;``  
 * 更改书写顺序，右``div``在上左``div``在下


## 使用``float+BFC``方案  
<hr style="border: 0.5px solid #ccc;">

上面的方法都需要通过右侧盒子的宽度，计算某个值，下面三种方法都是不需要计算的。只需要设置两个盒子之间的间隔。  

```
.wrapper-float-bfc{
    overflow: auto;
}

.wrapper-float-bfc .top-right{
    float: right;
    margin-left: 20px;
}

.wrapper-float-bfc .left{
    margin-right: 0;
    overflow: auto;
}
```

这个方案同样是利用了左侧浮动，但是左侧盒子通过``overflow: auto;``形成了BFC，因此左侧盒子不会与浮动的元素重叠。  
这种情况下，只需要为右侧的浮动盒子设置 ``margin-left``，就可实现两个盒子的距离了。而左侧盒子是``block``级别的，所以宽度能实现自适应。  
<strong>缺点：</strong>
 * 父元素需要清除浮动


## ``flex``方案  
<hr style="border: 0.5px solid #ccc;">

```
.wrapper-flex{
    display: flex;
    align-items: flex-start;
}

.wrapper-flex .bottom-right{
    flex: 0 0 auto;
}

.wrapper-flex .left{
    flex: 1 1 auto;
}
```

``flex``可以说是最好的方案了，代码少，使用简单。  
<strong>需要注意</strong>的是，``flex``容器的一个默认属性值：``align-items: stretch;``。这个属性导致了列等高的效果。  
为了让两个盒子高度自动，需要设置：``align-items: flex-start;``。  


## ``grid``方案  
<hr style="border: 0.5px solid #ccc;">

```
.wrapper-grid{
    display: grid;
    grid-template-columns: 2fr 200px;
    align-items: start;
}

.wrapper-grid .left,.wrapper-grid .bottom-right{
    box-sizing: border-box;
}

.wrapper-grid .bottom-right{
    grid-column: 2;
}

.wrapper-grid .left{
    grid-column: 1;
}
```

<strong>注意：</strong>

 * ``grid``布局也有列等高的默认效果。需要设置：``align-items: start``
 * ``grid``布局还有一个值得注意的小地方和``flex``不同：在使用``margin-right``的时候，``grid``布局默认是``box-sizing``设置的盒宽度之间的位置。而``flex``则是使用两个div的``border``或者``padding``外侧之间的距离  

<strong>链接：</strong>

 * 本文参考文章：<a href="https://segmentfault.com/a/1190000010698609" target="_blank">右边固定，左边自适应</a>
 * 参考文章的<a href="https://zhuqingguang.github.io/" target="_blank">作者</a>

