---
title: 04-spelling-correction
date: 2020-02-16 13:42:05
tags: nlp
categories: nlp
---

今天要介绍的是拼写检查，主要基于语言模型。

## Spelling Tasks

拼写检查任务主要包括两个方面，一方面是拼写错误侦测，另一方面是拼写错误修正。

拼写错误分为两种：

1.Non-word Errors：比如漏写导致的错误，graffe->giraffe。这类错误产生的词不存在。可以通过词典来进行侦测。找出与这些词相近的词，有两种方法，最小编辑路径法，信道法（Highest noisy channel probability）。

2.Real-word Errors：一种是打印错误，如 three->there顺序颠倒。另一种是认知错误，如piece->peace,too->two，发音相同导致的错误。这类错误产生的词是真实存在的，要先找到与这些词发音拼写相近的词形成候选集，在通过信道模型和分类器选择最好的候选词。

##  The Nosiy Channel Model of Spelling



The intuition of Noisy Model is that the words we see may be wrong words were produced through Noisy Channel。我们观察到的拼写错误的词，可以建立一个概率模型找到正确的词。
$$
\begin{aligned}
\hat{w} &=\underset{w \in V}{\operatorname{argmax}} P(w | x) \\
&=\underset{w \in V}{\operatorname{argmax}} \frac{P(x | w) P(w)}{P(x)} \\
&=\underset{w \in V}{\operatorname{argmax}} P(x | w) P(w)
\end{aligned}
$$
V是词表

### Non-word spellig error example

比如acress这个词我们要找到他的拼写正确的词

首先要产生候选集，候选集的产生又两种，一种是基于拼写上的相近，另一种是基于发音上的相近。

最小编辑距离又有四种类型：插入，删除，替换，相邻位置词的替换。插入删除 编辑距离为1，替换编辑距离为2

[![3pEYTg.md.png](https://s2.ax1x.com/2020/02/16/3pEYTg.md.png)](https://imgchr.com/i/3pEYTg)

大部分拼写错误的编辑距离都在1以内，几乎所有拼写错误都包含在编辑距离2.

会用到两种模型，一种是Language Model，另一种是Channel Model Probability，即正确的词产生某种拼写错误的概率。

$P(x | w)=$ probability of the edit	$w$是正确词 $x$是拼写错误的词

deletion/insertion/substitution/transposition)

拼写错误的四种情况可以产生对应的混淆矩阵

del[x,y]:  count(xy typed as x)

ins[x,y]:  count(x typed as xy)

sub[x,y]:  count(x typed as y)

trans[x,y]: count(xy typed as yx)

基于语料进行统计，比如有多少次xy被拼成了x啊，x被拼成了xy

![3pVtu6.png](https://s2.ax1x.com/2020/02/16/3pVtu6.png)

基于上图的混淆矩阵我们就可以计算对应错误的概率
$$
P(x | w)=\left\{\begin{array}{cl}
{\frac{\operatorname{del}\left[w_{i-1}, w_{i}\right]}{\operatorname{count}\left[w_{i-1}, w_{i}\right]},} & {\text { if deletion }} \\
{\frac{\operatorname{ins}\left[w_{i-1}, x_{i}\right]}{\operatorname{count}\left[w_{i-1}\right]},} & {\text { if insertion }} \\
{\frac{\operatorname{sub}\left[x_{i}, w_{i}\right]}{\operatorname{count}\left[w_{i}\right]},} & {\text { if substitution }} \\
{\frac{\operatorname{trans}\left[w_{i}, w_{i+1}\right]}{\operatorname{count}\left[w_{i}, w_{i+1}\right]},} & {\text { if transposition }}
\end{array}\right.
$$
这些概率可以用来对应候选集各个单词的概率，错误词对应的可能的真实词的概率

![3pZ1Z8.png](https://s2.ax1x.com/2020/02/16/3pZ1Z8.png)

再加上language model

![3pZyi4.png](https://s2.ax1x.com/2020/02/16/3pZyi4.png)

我们就可以找到最大概率的词，当然啦这只是基于当前词unigram

再看bigram

“a stellar and versatile **acress** whose combination of sass and glamour…”

•P(actress|versatile)=.000021

 P(whose|actress) = .0010

•P(across|versatile) =.000021

 P(whose|across) = .000006

基于bigram计算出对应前后两个词组成序列的概率

•P(“versatile actress whose”) = .000021*.0010 = 210 x10-10

•P(“versatile across whose”) = .000021*.000006 = 1 x10-10

再乘上对应的编辑距离概率，就能得出候选集各个词的概率

### Real-word spelling errors

就是这个词在字典中是存在的，但是并不是我们要的那个词。

•The design **an** construction of the system…

•Can they **lave** him my messages?

第一步也是产生候选集，for each word in sentence

因为这些词是真实存在与字典中的，所以词自身也是在候选集里，其他的还包括单一编辑距离词，发音相近的词等。

真实存在词拼写检查流程比较明显的特点就是它是基于整个句子，对每个词都产生候选集，将所有词进行组合，选择概率最大的那个序列路径的词

•Given a sentence w1,w2,w3,…,wn

•Generate a set of candidates for each word wi

•Candidate(w1) = {w1, w’1 , w’’1 , w’’’1 ,…}

•Candidate(w2) = {w2, w’2 , w’’2 , w’’’2 ,…}

•Candidate(wn) = {wn, w’n , w’’n , w’’’n ,…}

•Choose the sequence W that maximizes P(W)

下面这张图应该就比较好理解了

![3pmh5D.png](https://s2.ax1x.com/2020/02/16/3pmh5D.png)

很明显，two of the应该是计算后概率最大的路径。

也可以简化计算，每次只替换路径中的一个词，一般来说拼写错误只会拼错一个吧。

和non-word error相比,real-word error多了一个概率计算，no error $$\mathrm{P}(\mathrm{w} | \mathrm{w})$$
那么这个词本身就是正确的概率是多少呢？需要根据具体应用而定。假设你要做的应用，十个词就有一个是错误的，那么就是0.90，20个词里面有1个是错误的就是0.95.

看一下，现面的例子

![3puNkT.png](https://s2.ax1x.com/2020/02/16/3puNkT.png)

在unigram模型中，thew很有可能就是the拼错的，如果假设语料的拼写错误概率是0.9的情况下

## State-of-the-art Systems

不再简单的把先验概率乘上channel model产生的错误概率，将独立性假设转为概率不相同
$$
\hat{w}=\underset{w \in V}{\operatorname{argmax}} P(x | w) P(w)^{\lambda}
$$
针对non-word error，产生了上面的公式，$\lambda$可以通过development  test set 中学习出来。

### Phonetic error model

简单的来说就是加上发音来修正拼写错误，原来的拼写错误都是基于字母的拼写错误，现在把发音因素也考虑到拼写错误。比如下面的规则：

•“Drop duplicate adjacent letters, except for C.”

•“If the word begins with 'KN', 'GN', 'PN', 'AE', 'WR', drop the first letter.”

“Drop 'B' if after 'M' and if it is at the end of the 

去掉这些字母后，根据发音来进行spelling correction？？？

接着找到和拼写错误词发音相差1-2的最小编辑错误候选词

计算概率时对不同的编辑错误候选给与不同的权重。

### Improvement to channel model

更丰富的编辑距离，不仅仅是删除插入替换等

•ent-->ant

•ph-->f

•le-->al

将发音错误考虑到channel里面

### channel model

除了基于语料统计出插入删除替换产生的channel model，还可以考虑其他因素

•The source letter

•The target letter

•Surrounding letters

•The position in the word

•Nearby keys on the keyboard

•Homology on the keyboard

•Pronunciations

•Likely morpheme transformations

### classifier-based methods for real-word spelling correction

除了基于language model 、channel model的生成式模型，我们还可以利用特征输入分类器来进行拼写错误检查。

比如下面的例子

whether/weather

判断这个拼写错误到底是哪一个，我们考虑下面这些特征：
附近10个词内是否有cloudy，有的话那么很可能就是weather天气

后面 是不是接的 to VERB，是的话那么很有可能就是whether 是否

后面接的是不是 or not，是的话很有可能就是or not 是否