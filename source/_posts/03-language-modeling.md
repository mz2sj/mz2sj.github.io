---
title: 03-language-modeling
date: 2020-02-14 17:46:04
tags: [nlp]
categories: nlp
---

今天要学习的是语言模型，好多东西呀！

## 概念

老师介绍了一大堆，我的理解语言模型就是对语言的合理性进行建模。这种建模可以是一个完整的句子的可能性，也可以是一句话基于之前的词预测出下一个词的可能性。数学公式表达如下：

$\mathrm{P}(\mathrm{W})=\mathrm{P}\left(\mathrm{w}_{1}, \mathrm{W}_{2}, \mathrm{W}_{3}, \mathrm{W}_{4}, \mathrm{W}_{5} \dots \mathrm{W}_{\mathrm{n}}\right)$  一个句子出现的概率

$\mathrm{P}\left(\mathrm{w}_{\mathrm{n}} | \mathrm{w}_{1}, \mathrm{W}_{2} \dots \mathrm{W}_{\mathrm{n}-1}\right)$	基于之前词预测下一词

## 语言模型的计算

那么语言模型该如何计算呢？我们可以通过链式法则来计算一句话所有单词的联合概率


$$
P\left(w_{1} w_{2} \cdots w_{n}\right)=\prod_{i} P\left(w_{i} | w_{1} w_{2} \cdots w_{i-1}\right)
$$

举个例子

$\mathrm{P}\left(^{\prime \prime} \mathrm{its} \text { water is so transparent" }^{\prime \prime}\right)=$$\quad \mathrm{P}(\mathrm{its}) \times \mathrm{P}(\text { water } | \text { its }) \times \mathrm{P}(\text { is lits water })$$\quad \times \mathrm{P}(\text { so lits water is }) \times$P(transparent) its water is so$)$

这样一来，每个字都是依赖于之前所有的字，通过链式法则从第一个字一直传递到最后一个字，形成完整的一句话的概率。

但是随着句子长度增长，条件概率的计算变得不现实

$$
P(\text { the } | \text { its water is so transparent that })= Count (its water is so transparent that the) /Count (its water is so transparent that)
$$

我们通过对序列计数来计算条件概率，当句子过长时，句子序列数量增长，有些句子序列我们根本就没见过，会导致分母部分概率为0，导致无法计算。由此引入了马尔科夫假设，即一个事务的状态只与之前的状态有关。

整个序列的概率可以用如下公式估计
$$
P\left(w_{1} w_{2} \cdots w_{n}\right) \approx \prod P\left(w_{i} | w_{i-k} \cdots w_{i-1}\right)
$$
单个单词的条件概率估计
$$
P\left(w_{i} | w_{1} w_{2} \cdots w_{i-1}\right) \approx P\left(w_{i} | w_{i-k} \cdots w_{i-1}\right)
$$
当当前词的概率只与当前词有关，则是Unigram Model
$$
P\left(w_{1} w_{2} \cdots w_{n}\right) \approx \prod_{i} P\left(w_{i}\right)
$$
当当前词概率与前一个词有关时，则是Bigram Model
$$
P\left(w_{i} | w_{1} w_{2} \cdots w_{i-1}\right) \approx P\left(w_{i} | w_{i-1}\right)
$$
由此可以继续推出3-gram、4-gram Model等n-gram model模型。

n-gram模型基于条件独立假设，忽略了语言之间的依赖性，但我们仍使用它。

### 预测N-gram 概率

举个bigram例子
$$
P\left(w_{i} | w_{i-1}\right)=\frac{c\left(w_{i-1}, w_{i}\right)}{c\left(w_{i-1}\right)}
$$
在实际应用中，我们会加上句子的开始\<s\>和\</s\>结尾符。

```py
<s> I am Sam </s>
<s> Sam I am </s>
<s> I do not like green eggs and ham </s>
```

$$
\begin{array}{ll}
{P(\mathrm{I} |<\mathrm{s}>)=\frac{2}{3}=.67} & {P(\text { Sam } |<\mathrm{s})=\frac{1}{3}=.33 \quad P(\mathrm{am} | \mathrm{I})=\frac{2}{3}=.67} \\
{P(</ \mathrm{s}>| \text { Sam })=\frac{1}{2}=0.5} & {P(\text { Sam } | \mathrm{am})=\frac{1}{2}=.5 \quad P(\mathrm{do} | \mathrm{I})=\frac{1}{3}=.33}
\end{array}
$$

基于语料我们可以形成对应bigram表

![1jzkut.png](https://s2.ax1x.com/2020/02/14/1jzkut.png)

这种概率计算出来的概率会很小，可能会带来underflow问题，由此引出log函数将连乘转化为累加
$$
\log \left(p_{1} \times p_{2} \times p_{3} \times p_{4}\right)=\log p_{1}+\log p_{2}+\log p_{3}+\log p_{4}
$$

## 评价语言模型 perplexity

除了可以根据任务评价语言模型外，最直接的语言模型评价方法就是perplexity

一个好的语言模型应该能给真实的句子更高的概率，我们可以据此来评价语言模型。将模型在测试集上的概率的倒数n次幂后作为perplexity，bigrams公式如下
$$
\operatorname{PP}(W)=\sqrt[N]{\prod_{i=1}^{N} \frac{1}{P\left(w_{i} | w_{i-1}\right)}}
$$
困惑度越低代表模型越好

## Smoothing 平滑

当训练集和测试集很相似的时候，n-grams模型可以取得很好效果。但是现实中，训练集和测试集常有差别，导致在测试集中出现许多在训练集中没见过的东西。没见过的东西概率是0，那么测试集中原本是正确的句子可能被预测为错误，而且概率为0，导致无法计算困惑度。

### 劫富济贫

将一些概率较高的gram分配给一些未出现过的gram

![1vFolD.png](https://s2.ax1x.com/2020/02/14/1vFolD.png)

### Add-one estimation

对所有词都加一，那么分母就要机上词表大小V
$$
P_{\text {Add-1}}\left(w_{i} | w_{i-1}\right)=\frac{c\left(w_{i-1}, w_{i}\right)+1}{c\left(w_{i-1}\right)+V}
$$
![1vks4P.png](https://s2.ax1x.com/2020/02/14/1vks4P.png)

Add-one方法显得十分粗糙，所以并没有用在N-grams中，但在文本分类和0的数目不是很多的任务中会用到。

也可以不加1，加k，由此引出下列模型
$$
\begin{aligned}
&P_{\text {Add }-k}\left(w_{i} | w_{i-1}\right)=\frac{c\left(w_{i-1}, w_{i}\right)+k}{c\left(w_{i-1}\right)+k V}\\
&P_{\text {Add-k}}\left(w_{i} | w_{i-1}\right)=\frac{c\left(w_{i-1}, w_{i}\right)+m\left(\frac{1}{V}\right)}{c\left(w_{i-1}\right)+m}
\end{aligned}
$$
保证分子加上的和和分母加上的相等即可，也可加上当前词的概率
$$
\begin{aligned}
&P_{\text {Add-k}}\left(w_{i} | w_{i-1}\right)=\frac{c\left(w_{i-1}, w_{i}\right)+m\left(\frac{1}{V}\right)}{c\left(w_{i-1}\right)+m}\\
&P_{\text {UnigramPrior }}\left(w_{i} | w_{i-1}\right)=\frac{c\left(w_{i-1}, w_{i}\right)+m P\left(w_{i}\right)}{c\left(w_{i-1}\right)+m}
\end{aligned}
$$


## 模型的融合

### Backoff 回退

试一试那种语言模型好，谁好用谁。例如计算trigram时，预料中没有这个trigram，就用bigram来代替，bigram没有就用unigram代替。

### Interpolation 插值

将 unigram，bigram，trigram混用，通常会表现的更好。

简单的interpolation
$$
\begin{aligned}
\hat{P}\left(w_{n} | w_{n-1} w_{n-2}\right)=& \lambda_{1} P\left(w_{n} | w_{n-1} w_{n-2}\right) \\
&+\lambda_{2} P\left(w_{n} | w_{n-1}\right) \\
&+\lambda_{3} P\left(w_{n}\right)
\end{aligned}
$$

$$
\sum_{i} \lambda_{i}=1
$$

基于上文的interpolation
$$
\begin{aligned}
\hat{P}\left(w_{n} | w_{n-2} w_{n-1}\right)=& \lambda_{1}\left(w_{n-2}^{n-1}\right) P\left(w_{n} | w_{n-2} w_{n-1}\right) \\
&+\lambda_{2}\left(w_{n-2}^{n-1}\right) P\left(w_{n} | w_{n-1}\right) \\
&+\lambda_{3}\left(w_{n-2}^{n-1}\right) P\left(w_{n}\right)
\end{aligned}
$$


我们要找到能使概率最大的lambda
$$
\log P\left(w_{1} \ldots w_{n} | M\left(\lambda_{1} \ldots \lambda_{k}\right)\right)=\sum_{i} \log P_{M\left(\lambda_{1} \ldots \lambda_{k}\right)}\left(w_{i} | w_{i-1}\right)
$$
对于大规模的n-gram模型，如果某个词序列个数为0，导致n-gram为0，则可以降低n-gram阶数，计算之前的n-gram概率。
$$
S\left(w_{i} | w_{i-k+1}^{i-1}\right)=\left\{\begin{array}{cc}
{\frac{\operatorname{count}\left(w_{i-k+1}^{i}\right)}{\operatorname{count}\left(w_{i-k+1}^{i-1}\right)}} & {\text { if } \operatorname{count}\left(w_{i-k+1}^{i}\right)>0} \\
{0.4 S\left(w_{i} | w_{i-k+2}^{i-1}\right)} & {\text { otherwise }}
\end{array}\right.
$$

## 高级平滑

### Absolute Discounting Interpolation

$$
P_{\text {AbsoluteDiscounting }}\left(w_{i} | w_{i-1}\right)=\frac{c\left(w_{i-1}, w_{i}\right)-d}{c\left(u_{i}\right)}+\lambda\left(\stackrel{d}{w}_{i-1}\right) P(w)
$$

将我们频率出现较高的词减去一部分，分给较低的一部分。d通常取0.75或者比较小的数

### Kneser-Ney Smotting

 I used to eat Chinese food with ______ instead of knife and fork. 

当我们在一句话中预测未知词，我们除了要考虑词的unigram，还要考虑这个词的bigram，它是否适合和别的词组成词，即一种连续性。
$$
P_{\text {CONTTNUATION }}(w)=\frac{\left|\left\{w_{i-1}: c\left(w_{i-1}, w\right)>0\right\}\right|}{|\left\{\left(w_{j-1}, w_{j}\right): c\left(w_{j-1}, w_{j}\right)>0\right\}}
$$
上面时w这个词在语料中产生的bigram数量，下面是语料中所有词的bigram数量
$$
P_{\text {CONTTNUATON }}(w)=\frac{\left|\left\{w_{i-1}: c\left(w_{i-1}, w\right)>0\right\}\right|}{\sum_{w^{\prime}}\left|\left\{w_{i-1}^{\prime}: c\left(w_{i-1}^{\prime}, w^{\prime}\right)>0\right\}\right|}
$$
将$P_{\text {CONTINUATION }}(w)$引入之前定义的abosolute discounting
$$
P_{K N}\left(w_{i} | w_{i-1}\right)=\frac{\max \left(c\left(w_{i-1}, w_{i}\right)-d, 0\right)}{c\left(w_{i-1}\right)}+\lambda\left(w_{i-1}\right) P_{\text {CONTINUATION}}\left(w_{i}\right)
$$
max是为了使分子都大于0
$$
\lambda\left(w_{i-1}\right)=\frac{d}{c\left(w_{i-1}\right)}\left|\left\{w: c\left(w_{i-1}, w\right)>0\right\}\right|
$$
$\lambda$取值，只有当$c\left(w_{i-1}, w_{i}\right)$>0时才会减去d，这里又乘上减去d的次数。

平滑算法太复杂啦，今天先到这儿，我还要再理解理解。