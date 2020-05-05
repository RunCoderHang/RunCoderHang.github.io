---
title:  LaTex Test
date:   2020-05-04 20:00:00
categories: 学习总结
mathjax: true
---



测试使用 `markdown` 中书写 `Latex` 数学公式。 **PAT** 中总是有一堆数学公式，GitHub上不支持，结果在博客中发现了这个功能。

多刷新几次，总会渲染成功的。

## 1. 呈现位置

### 1.1 行内公式：$...$

- 正文(inline)中的LaTeX公式用 `$...$` 定义

是的，我就是行内公式：$e^{x^2}\neq{e^x}^2$，排得OK吗？

$\sum_{i=0}^N\int_{a}^{b}g(t,i)\text{d}t$

显示复杂度： $O(logn)$ 、 $O(log^2n)$ 、 $O(n)$ 、 $O(nlogn)$ 、 $O(n^2)$ 、 $O(n^3)$ 、 $O(2^n)$


### 1.2 块公式：$$...$$

- 单独显示(display)的LaTeX公式用 `$$...$$` 定义，此时公式居中并放大显示

$$e^{x^2}\neq{e^x}^2$$

$$\sum_{i=0}N\int_{a}{b}g(t,i)\text{d}t$$

来个 *"复杂点"* 的:

$$H(D_2) = -(\frac{2}{4}\ log_2 \frac{2}{4} + \frac{2}{4}\ log_2 \frac{2}{4}) = 1$$


## 2. 希腊字母

|    显示    |   命令   |   显示   |  命令  |
|------------|----------|----------|--------|
| $\alpha$   | \alpha   | $\beta$  | \beta  |
| $\gamma$   | \gamma   | $\delta$ | \delta |
| $\epsilon$ | \epsilon | $\zeta$  | \zeta  |
| $\eta$     | \eta     | $\theta$ | \theta |
| $\iota$    | \iota    | $\kappa$ | \kappa |
| $\lambda$  | \lambda  | $\mu$    | \mu    |
| $\nu$      | \nu      | $\xi$    | \xi    |
| $\pi$      | \pi      | $\rho$   | \rho   |
| $\sigma$   | \sigma   | $\tau$   | \tau   |
| $\upsilon$ | \upsilon | $\phi$   | \phi   |
| $\chi$     | \chi     | $\psi$   | \psi   |
| $\omega$   | \omega   |          |        |


## 3. 字母修饰


### 3.1 上下标

* 上标： `^`
* 下标： `_`

`C_n^2` : $C_n^2$


### 3.2 矢量

`\vec a` : $\vec a$

`\overrightarrow{xy}` : $\overrightarrow{xy}$


### 3.3 字体

* Typewriter: `\mathtt{ABCDEFGHIJKLMNOPQRSTUVWXZ}`
    - $\mathtt{ABCDEFGHIJKLMNOPQRSTUVWXZ}$
* Blackboard Bold: `\mathbb{ABCDEFGHIJKLMNOPQRSTUVWXZ}`
    - $\mathbb{ABCDEFGHIJKLMNOPQRSTUVWXZ}$
* Sans Serif: `\mathsf{ABCDEFGHIJKLMNOPQRSTUVWXZ}`
    - $\mathsf{ABCDEFGHIJKLMNOPQRSTUVWXZ}}$


### 3.4 分组

* 使用 `{}` 将具有相同等级的内容扩入其中，成组处理
* 例如： `10^{10}`: $10^{10}$ ， 然而 `10^10`: $10^10$


### 3.5 括号

* 小括号： `()` 呈现为 $()$
* 中括号： `[]` 呈现为 $[]$
* 尖括号： `\langle...\rangle` 呈现为 $\langle...\rangle$
* 使用 `\left(或\right)` 使符号大小与邻近的公式相适应；该语句适用于所有括号类型
    - `(\frac{x}{y})` 呈现为 $(\frac{x}{y})$
    - `\left(\frac{x}{y}\right)` 呈现为 $\left(\frac{x}{y}\right)$


### 3.6 求和、极限和积分

* 求和： `\sum`
    - 例如： `\sum_{i=1}^n{a_i}` 呈现为 $\sum_{i=1}^n{a_i}$
* 极限： `\lim_{x\to 0}` 呈现为 $\lim_{x\to 0}$
* 积分： `\int`
    - 例如： `\int_0^\infty{fxdx}` 呈现为 $\int_0^\infty{fxdx}$


### 3.7 分式与根式

* 分式： `\frac{公式1}{公式2}` 呈现为 $\frac{公式1}{公式2}$
* 根式： `\sqrt{x}{y}` 呈现为 $\sqrt{x}{y}$


### 3.8 特殊函数

* `\函数名`
* 例如： `\sin x` 、 `\ln x` 、 `\max (A,B,C)` 呈现为 $\sin x$ 、 $\ln x$ 、 $\max (A,B,C)$


### 3.9 特殊符号

|     显示    |    命令   |      显示     |     命令    |
|-------------|-----------|---------------|-------------|
| $\infty$    | \infty    | $\varnothing$ | \varnothing |
| $\cup$      | \cup      | $\forall$     | \forall     |
| $\cap$      | \cap      | $\exists$     | \exists     |
| $\subset$   | \subset   | $\lnot$       | \lnot       |
| $\subseteq$ | \subseteq | $\nabla$      | \nabla      |
| $\in$       | \in       | $\partial$    | \partial    |
| $\notin$    | \notin    |               |             |



### 3.10 空格

* LaTeX语法本身会忽略空格的存在
* 小空格： `a\ b` 呈现为 $a\ b$
* 4格空格： `a\quad b` 呈现为 $a\quad b$


## 4. 矩阵

### 4.1 基本语法

* 起始标记： `\begin{matrix}`
* 结束标记： `\end{matrix}` 
* 每一行末尾标记 `\\` ，行间元素之间以 `&` 分隔



```
$$
\begin{matrix}
1&0&0\\
0&1&0\\
0&0&1\\
\end{matrix}
$$
```

呈现为

$$
\begin{matrix}
1&0&0\\
0&1&0\\
0&0&1\\
\end{matrix}
$$

### 4.2 矩阵边框

* 替换起始的 `matrix`
* `pmatrix` ：小括号边框
* `bmatrix` ：中括号边框
* `Bmatrix` ：大括号边框
* `vmatrix` ：单竖线边框
* `Vmatrix` ：双竖线边框

```
$$
\begin{pmatrix}
1 & a_1 & a_1^2 & \cdots & a_1^n \\
1 & a_2 & a_2^2 & \cdots & a_2^n \\
\vdots & \vdots & \vdots & \ddots & \vdots \\
1 & a_m & a_m^2 & \cdots & a_m^n \\
\end{pmatrix}
$$
```

呈现为

$$
\begin{pmatrix}
1 & a_1 & a_1^2 & \cdots & a_1^n \\
1 & a_2 & a_2^2 & \cdots & a_2^n \\
\vdots & \vdots & \vdots & \ddots & \vdots \\
1 & a_m & a_m^2 & \cdots & a_m^n \\
\end{pmatrix}
$$


### 4.3 省略元素

* 横省略号： `\cdots`
* 竖省略号： `\vdots`
* 斜省略号： `\ddots`

```
$$
\begin{bmatrix}
{a_{11}}&{a_{12}}&{\cdots}&{a_{1n}}\\
{a_{21}}&{a_{22}}&{\cdots}&{a_{2n}}\\
{\vdots}&{\vdots}&{\ddots}&{\vdots}\\
{a_{m1}}&{a_{m2}}&{\cdots}&{a_{mn}}\\
\end{bmatrix}
$$
```

呈现为

$$
\begin{bmatrix}
{a_{11}}&{a_{12}}&{\cdots}&{a_{1n}}\\
{a_{21}}&{a_{22}}&{\cdots}&{a_{2n}}\\
{\vdots}&{\vdots}&{\ddots}&{\vdots}\\
{a_{m1}}&{a_{m2}}&{\cdots}&{a_{mn}}\\
\end{bmatrix}
$$



### 4.4 阵列

* 需要array环境：起始、结束以 `{array}` 声明
* 对齐方式：在 `{array}` 后以 `{}` 逐行统一声明
    - 左对齐： `l` ; 居中： `c` ; 右对齐： `r`
    - 竖直线： 在声明对齐方式时，插入 `|` 建立竖直线
* 插入水平线： `\hline`

```
$$
\begin{array}{c|lll}
{↓}&{a}&{b}&{c}\\
\hline
{R_1}&{c}&{b}&{a}\\
{R_2}&{b}&{c}&{c}\\
\end{array}
$$
```

呈现为

$$
\begin{array}{c|lll}
{↓}&{a}&{b}&{c}\\
\hline
{R_1}&{c}&{b}&{a}\\
{R_2}&{b}&{c}&{c}\\
\end{array}
$$


### 4.5 方程组

* 需要cases环境：起始、结束处以 `{cases}` 声明

```
$$
\begin{cases}
a_1x+b_1y+c_1z=d_1\\
a_2x+b_2y+c_2z=d_2\\
a_3x+b_3y+c_3z=d_3\\
\end{cases}
$$
```

呈现为

$$
\begin{cases}
a_1x+b_1y+c_1z=d_1\\
a_2x+b_2y+c_2z=d_2\\
a_3x+b_3y+c_3z=d_3\\
\end{cases}
$$


