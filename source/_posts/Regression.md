---
title: Regression
date: 2021-04-06 21:32:09
tags: [算法]
categories: [算法,数据分析]
---

em...,我又来立flag了，之前买了《统计学习方法》这本书，本以为会发奋读书，没想到也是半途而废。现在想想还是得学点算法类的知识，

求求你了，孟镇，做个人吧，能写出来嘛？

求求你，抄别人的也抄出来吧！

## 线性回归

数据集：

假设数据集为:

$\mathcal{D}=\left\{\left(x_{1}, y_{1}\right),\left(x_{2}, y_{2}\right), \cdots,\left(x_{N}, y_{N}\right)\right\}$

后面我们记:

$X=\left(x_{1}, x_{2}, \cdots, x_{N}\right)^{T}, Y=\left(y_{1}, y_{2}, \cdots, y_{N}\right)^{T}$

线性回归假设:

$f(w)=w^{T} x$

### 最小二乘法

对这个问题, 采用二范数定义的平方误差来定义损失函数:

$L(w)=\sum_{i=1}^{N}\left\|w^{T} x_{i}-y_{i}\right\|_{2}^{2}$

展开得到：

$\begin{aligned}
L(w) &=\left(w^{T} x_{1}-y_{1}, \cdots, w^{T} x_{N}-y_{N}\right) \cdot\left(w^{T} x_{1}-y_{1}, \cdots, w^{T} x_{N}-y_{N}\right)^{T} \\
&=\left(w^{T} X^{T}-Y^{T}\right) \cdot(X w-Y)=w^{T} X^{T} X w-Y^{T} X w-w^{T} X^{T} Y+Y^{T} Y \\
&=w^{T} X^{T} X w-2 w^{T} X^{T} Y+Y^{T} Y
\end{aligned}$

最小化这个值的  $\hat{w}  $,这里注意对用变量的求导情况：

$\begin{aligned}
\hat{w}=\operatorname{argmin}_{w} L(w) & \longrightarrow \frac{\partial}{\partial w} L(w)=0 \\
& \longrightarrow 2 X^{T} X \hat{w}-2 X^{T} Y=0 \\
& \longrightarrow \hat{w}=\left(X^{T} X\right)^{-1} X^{T} Y=X^{+} Y
\end{aligned}$

这个式子中  $\left(X^{T} X\right)^{-1} X^{T} $ 又被称为伪逆。对于行满秩或者列满秩的  X , 可以直接求解, 但是对于非满秩的样本集合，需要使用奇异值分解 (SVD) 的方法, 对  X  求奇异值分解, 得到

$X=U \Sigma V^{T}$

于是:

$X^{+}=V \Sigma^{-1} U^{T}$

这里的$X^+$怎么得来的还不清楚

### 几何想象法

我们通过对样本$X$的组合，也就是$x_1,x_2,...,x_n$获得预测值, $Y=\left(y_{1}, y_{2}, \cdots, y_{N}\right)^{T}$是$n*1$的向量，则我们希望存在一个向量$m*1$的向量$\beta$ ，使得$X^{T}\beta$与$Y$之间的距离越小越好，那么$X^{T}\beta$与$Y$之间的差就与$X^{T}$的基本空间垂直，即：

$X^{T} \cdot(Y-X \beta)=0 \longrightarrow \beta=\left(X^{T} X\right)^{-1} X^{T} Y$



### 噪声先验高斯分布的MLE

对于一维的情况，记  $y=w^{T} x+\epsilon$, $\epsilon \sim \mathcal{N}\left(0, \sigma^{2}\right)$ , 那么 $ y \sim \mathcal{N}\left(w^{T} x, \sigma^{2}\right)$  。代入极大似然估计中：

$\begin{aligned}
L(w)=\log p(Y \mid X, w) &=\log \prod_{i=1}^{N} p\left(y_{i} \mid x_{i}, w\right) \\
&=\sum_{i=1}^{N} \log \left(\frac{1}{\sqrt{2 \pi \sigma}} e^{-\frac{\left(y_{i}-u^{T} x_{i}\right)^{2}}{2 \sigma^{2}}}\right) \\
\underset{w}{\operatorname{argmax}} L(w) &=\underset{w}{\operatorname{argmin}} \sum_{i=1^{N}}\left(y_{i}-w^{T} x_{i}\right)^{2}
\end{aligned}$

这个表达式和最小二乘估计得到的结果一样。

### 权重先验为高斯分布的MAP-最大后验估计

取先验分布  $w \sim \mathcal{N}\left(0, \sigma_{0}^{2}\right) $, $\epsilon \sim \mathcal{N}\left(0, \sigma^{2}\right)$  ,$y \mid x ; w \sim\left(w^{\top} x, \sigma^{2}\right)$。于是：

$\begin{aligned}
\hat{w}=\operatorname{argmax}_{w} p(w \mid Y) &=\underset{w}{\operatorname{argmax}} p(Y \mid w) p(w) \\
&=\underset{w}{\operatorname{argmax}} \log p(Y \mid w) p(w) \\
&=\underset{w}{\operatorname{argmax}}(\log p(Y \mid w)+\log p(w)) \\
&=\underset{w}{\operatorname{argmin}}\left[\left(y-w^{T} x\right)^{2}+\frac{\sigma^{2}}{\sigma_{0}^{2}} w^{T} w\right]
\end{aligned}$

这里省略了$X$,$ p(Y)$  和  $w$ 没有关系:

$p(w \mid Y)=\frac{p(w, Y)}{p(Y)}=\frac{p(Y|w| p(w)}{p(Y)}$

$ p(Y)$是已有先验数据和$w$无关。

### 正则化

正则化的两种方式：

$\begin{array}{l}
L 1: \underset{w}{\operatorname{argmin}} L(w)+\lambda\|w\|_{1}, \lambda>0 \\
L 2: \operatorname{argmin}_{w} L(w)+\lambda\|w\|_{2}^{2}, \lambda>0
\end{array}$

#### L1正则化 Lasso

L1正则化可以引起稀疏解
从最小化损失的角度看，由于  $\mathrm{L} 1$  项求导在O附近的左右导数都不是0, 因此更容易取到0
解。
从另一个方面看, L1 正则化相当于:

$\begin{array}{l}
\underset{w}{\operatorname{argmin}} L(w) \\
\text { s.t. }\|w\|_{1}<C
\end{array}$

我们已经看到平方误差损失函数在  $w $空间是一个植球, 因此上式求解就是尼球和
$ \|w\|_{1}=C $ 的切点, 因此更容易相切在坐标轴上。

这一块没太懂

#### L2正则化 Ridge

$\begin{aligned}
\hat{w}=\operatorname{argmin}_{w} L(w)+\lambda w^{T} w & \longrightarrow \frac{\partial}{\partial w} L(w)+2 \lambda w=0 \\
& \longrightarrow 2 X^{T} X \hat{w}-2 X^{T} Y+2 \lambda \hat{w}=0 \\
& \longrightarrow \hat{w}=\left(X^{T} X+\lambda \mathbb{I}\right)^{-1} X^{T} Y
\end{aligned}$

可以看到，这个正则化参数和前面的 MAP 结果不谋而合。利用2范数进行正则化不仅可 以是模型选择  $w$  较小的参数，同时也避免 $X^{T}X$不可逆的问题。

### 逻辑回归