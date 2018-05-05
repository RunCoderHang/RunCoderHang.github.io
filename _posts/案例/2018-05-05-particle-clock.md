---
layout: blog
domo: true
new: true
ico: bg-ico-web
title:  初步实现粒子时钟
create: 原创
tags:
- js
- canvas
- 数组
background: blue
background-image: 
date:   2018-05-05 15:41:00
category: 案例
---

## 前言  

学习了使用``canvas``和``js``实现粒子时钟的效果。知识点有些多，所以在此整理一下。  
可查看<a href="https://hangforfreedom.github.io/some-cases/demo-6/demo.html" target="_blank">案例</a>

## 思路

<b>一、``js``拓印方法</b>    

要在canvas上面进行图形绘制，用到了拓印(ta yin)的方法。如何使用``js``的进行拓印？有两种方法：

 - <b>数组</b> 特点:使用最多，实现起来简单、快捷
 - <b>矩阵</b> 特点:需要一定的数学理论基础，实现起来有难度

<b>二、数组怎么拓印？</b>  

首先需要考虑如何建立数组，也就是确立数组的维度：

 - 要绘制的数字 （一维）
 - 要绘制数字的长度 （二维）
 - 要绘制的行数和列数 （三维）

所以我们确立<font color="red">三维数组，包含0~9以及冒号(:)的二维点阵</font>。每个数字的点阵表示是7*10大小的二维数组。  

    通过遍历数字点阵的二维数组，当该位置的值为1时，则绘制一个粒子，否则不绘制

<b>三、借助``canvas``进行绘制</b>  

<font color="red">需要知道</font>：一个粒子是一个圆形，圆形之间必须要有间距否则会紧挨在一起，所以需要一个矩形将圆形包裹起来并且矩形和圆形留有间距。不废话，看图！

<img src="https://hangforfreedom.github.io/thumbnails/canvasTime.png" alt="canvasTime">

确立画圆的方法：``arc( x, y, r, 0, 2*PI, false);``。可以画圆后再确立圆的半径``r``以及每个圆的中心坐标  
来吧，我们要开始做数学题了(微笑)

 - 求圆的半径

>设圆的半径为``r``，则一个圆的宽高(直径)为``2r``。矩形和圆形的间距为1，则一个矩形的宽高为``2(r+1)``。
  解：因为在 10*7 的二维点阵，所以整个数字的宽度为``14(r+1)``，高度为``20(r+1)``
      假设数字的高度为100，则``100=20(r+1)``，所以``r=4``

 - 求圆的中心坐标

>设``x轴``里有``i``个圆，设``y轴``里有``j``个圆
  解：确立圆形的x坐标： 
      第一个圆：r + 1
      第二个圆：2（r + 1） + r + 1
      第三个圆：4（r + 1） + r + 1
      ...
      综上，得到 j * 2（r + 1） + r + 1
      确立圆形的x坐标： 
      第一个圆：r + 1
      第二个圆：2（r + 1） + r + 1
      第三个圆：4（r + 1） + r + 1
      ...
      综上，得到 i * 2（r + 1） + r + 1
  

## 生成点阵数字

```html
<canvas id="canvas">当前浏览器不支持canvas，请更换浏览器后再试</canvas>
<script>
    //时钟的构造函数 盛放私有属性
    function Clock(){
        //画笔 创造2d绘画空间
        this.cxt = canvas.getContext('2d'); 
        //声明canvas的宽高
        canvas.width = 700;
        canvas.height = 100;
        this.H = canvas.height;
        this.W = canvas.width;
    }
    //对象的原型， 盛放公共的方法
    Clock.prototype = {
        init: function(index,num){
            //设置粒子的半径
            var r = (this.H / 20) - 1;
            var cxt = this.cxt;
            //渲染的数字，生成点阵数字
            for(var i=0; i<digit[num].length; i++){ 
                for(var j=0; j<digit[num][i].length; j++){
                    if(digit[num][i][j] == 1){
                        cxt.fillStyle = '#000';
                        cxt.beginPath();
                        cxt.arc(index*14*(r+2) + j*2*(r+1)+r+1,i*2*(r+1)+r+1,r,0,Math.PI*2,false);
                        cxt.closePath();
                        cxt.fill();
                    }
                }

            }
        },
    }
</script>
```


## 时钟实现

在上一步的点阵数字的基础上，实现一个粒子时钟。将时钟实现的函数命名为``getTime()``，时钟实现由获取时间数据和渲染时钟两部分组成

 - 时间数据

最简单的时钟形式由两位的小时、两位的分钟和两位的秒钟组成，中间用冒号隔开。通过日期对象``Date()``来获取当前时间，以及当前的小时、分钟和秒钟。但是，最终需要得到的是数字表示的时钟

比如``12:02:36``的时间数据的表示形式为``data[1,2,10,0,2,10,3,6]``

 - 渲染时钟

获取到时间数据后，通过循环来渲染时钟中的每一个数字。此时，有一个需要改变的地方是``arc()``函数中的x坐标，否则它们将叠加在一起

为了将时钟数字表示更加清晰在每个数字之间增加一定的间距。每个数字的宽度是``14(R+1)``，假设``data``数组中7个数字的索引为``index``，则每个数字的起始x坐标可以等于``14(R+2)*index``

最后通过定时器每间隔一段时间后更新时间

```html
<canvas id="canvas">当前浏览器不支持canvas，请更换浏览器后再试</canvas>
<script>
    //时钟的构造函数 盛放私有属性
    function Clock(){
        //画笔 创造2d绘画空间
        this.cxt = canvas.getContext('2d'); 
        //声明canvas的宽高
        canvas.width = 700;
        canvas.height = 100;
        this.H = canvas.height;
        this.W = canvas.width;
    }
    //对象的原型， 盛放公共的方法
    Clock.prototype = {
        init: function(index,num){
            //设置粒子的半径
            var r = (this.H / 20) - 1;
            var cxt = this.cxt;
            //渲染的数字，生成点阵数字
            for(var i=0; i<digit[num].length; i++){ 
                for(var j=0; j<digit[num][i].length; j++){
                    if(digit[num][i][j] == 1){
                        cxt.fillStyle = '#000';
                        cxt.beginPath();
                        cxt.arc(index*14*(r+2) + j*2*(r+1)+r+1,i*2*(r+1)+r+1,r,0,Math.PI*2,false);
                        cxt.closePath();
                        cxt.fill();
                    }
                }

            }
        },
        getTime: function(){
            //存储时间数据
            var data = [];
            //存储时间数字，由十位小时、个位小时、冒号、十位分钟、个位分钟、冒号、十位秒钟、个位秒钟这7个数字组成
            var temp = /(\d)(\d):(\d)(\d):(\d)(\d)/.exec(new Date());
            data.push(temp[1],temp[2],10,temp[3],temp[4],10,temp[5],temp[6]);
            //重置画布高低，达到清空画布的效果
            canvas.height = 100; 
            //渲染时钟
            for(var i=0; i<data.length; i++){
                clock.init(i,data[i]);
            }
            
        }
    }

    var clock = new Clock();
    //优化
    clearInterval(oTimer); 
    var oTimer = setInterval(function(){
        clock.getTime();
    },50);
</script>
```


<canvas id="canvas" style="display: block;"></canvas>
<script>
    function Clock(){
            //画笔 创造2d绘画空间
            this.cxt = canvas.getContext('2d'); 
            //声明canvas的宽高
            canvas.width = 500;
            canvas.height = 70;
            this.H = canvas.height;
            this.W = canvas.width;
        }
        //对象的原型， 盛放公共的方法
        Clock.prototype = {
            init: function(index,num){
                //设置粒子的半径
                var r = (this.H / 20) - 1;
                var cxt = this.cxt;
                //渲染的数字，生成点阵数字
                for(var i=0; i<digit[num].length; i++){ 
                    for(var j=0; j<digit[num][i].length; j++){
                        if(digit[num][i][j] == 1){
                            cxt.fillStyle = '#000';
                            cxt.beginPath();
                            cxt.arc(index*14*(r+2) + j*2*(r+1)+r+1,i*2*(r+1)+r+1,r,0,Math.PI*2,false);
                            cxt.closePath();
                            cxt.fill();
                        }
                    }

                }
            },
            getTime: function(){
                //存储时间数据
                var data = [];
                var temp = /(\d)(\d):(\d)(\d):(\d)(\d)/.exec(new Date());
                data.push(temp[1],temp[2],10,temp[3],temp[4],10,temp[5],temp[6]);
                //重置画布高低，达到清空画布的效果
                canvas.height = 70; 
                //渲染时钟
                for(var i=0; i<data.length; i++){
                    clock.init(i,data[i]);
                }
                
            }
        }
        var clock = new Clock();
        clearInterval(oTimer); //优化
        var oTimer = setInterval(function(){
            clock.getTime();
        },50);
</script>