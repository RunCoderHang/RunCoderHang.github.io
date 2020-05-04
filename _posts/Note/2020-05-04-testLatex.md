---
title:  Test Latex
date:   2020-05-04 20:00:00
categories: 生活随笔
mathjax: true
---



### Md2All 简介

Markdown排版利器，支持 **"一键排版"** 、自定义css、80多种代码高亮。
能让Markdown内容，无需作任何调整就能**一键复制**到微信公众号、博客园、掘金、知乎、csdn、51cto、wordpress、hexo。。。等平台。
支持把图片自动上传到云图床;
支持Latex数学公式在公众号等平台完美显示;
支持生成带样式的html文件;
甚至支持直接用原生的html,css排版。

#### 详细教程

[Md2All详细教程,请参考：https://www.cnblogs.com/garyyan/p/8329343.html](https://www.cnblogs.com/garyyan/p/8329343.html )



### 代码块显示效果
注：markdown对代码块的语法是开始和结束行都要添加：\`\`\`,其中 \` 为windows键盘左上角那个，如下：

```java
public class MyActivity extends AppCompatActivity {
@Override  //override the function
    protected void onCreate(@Nullable Bundle savedInstanceState) {
       super.onCreate(savedInstanceState);
       try {
            OkhttpManager.getInstance().setTrustrCertificates(getAssets().open("mycer.cer");
            OkHttpClient mOkhttpClient= OkhttpManager.getInstance().build();
        } catch (IOException e) {
            e.printStackTrace();
        }
}
```
要精确指定语言（如：`java,cpp,css,xml,javascript,python,php,go,kotlin,lua,objectivec`等等）时，在头部直接指定，如：\`\`\`javascript，如下：

```javascript
function DisplayWindowSize(){

  var w=window.innerWidth
  || document.documentElement.clientWidth
  || document.body.clientWidth;
}
```


### Latex数学公式(能正确复制到公众号等平台):

“复制”时会自动把Latex数学公式转换为图片，并自动上传到云图床（如果在“图片”设置了“...,自动上传到云图床”）。

[请参考：Md2All,让公众号完美显示Latex数学公式](https://www.cnblogs.com/garyyan/p/9228994.html)

#### 行内公式：$...$

是的，我就是行内公式：$e^{x^2}\neq{e^x}^2$，排得OK吗？

#### 块公式：$$...$$

$$e^{x^2}\neq{e^x}^2$$

来个 *"复杂点"* 的:

$$H(D_2) = -(\frac{2}{4}\ log_2 \frac{2}{4} + \frac{2}{4}\ log_2 \frac{2}{4}) = 1$$

矩阵：

$O(n^2)$

$$
        \begin{pmatrix}
        1 & a_1 & a_1^2 & \cdots & a_1^n \\
        1 & a_2 & a_2^2 & \cdots & a_2^n \\
        \vdots & \vdots & \vdots & \ddots & \vdots \\
        1 & a_m & a_m^2 & \cdots & a_m^n \\
        \end{pmatrix}
$$

对应“一键排版”的css样式关键字为：`.katex`

