---
title: 02 basic text processing
date: 2020-02-12 20:45:27
tags: [nlp,正则表达式]
categories: nlp
---

今天学习正则表达式的使用，后面还有一个小作业要做，开始啦！

# 正则表达式

## Disjunctions 析取

我也不知道中文该怎么说，暂定析取吧。disjunction用 **[]** 表示，表示选取括号中表示的一个字符。下面举个例子

| Pattern      | Matches                                             |
| ------------ | --------------------------------------------------- |
| [wW]oodchunk | Woodchunk,woodchunk                                 |
| [123456789]  | any digit(由于匹配是贪心的，所以只要是数字都会匹配) |

像数字每次都要把十个阿拉伯数字都写出来太麻烦了，所以有了range用法

| Pattern | Matches               |
| ------- | --------------------- |
| [A-Z]   | An uppeer case letter |
| [a-z]   | A lowwer case letter  |
| [0-9]   | A single digit        |

## Negation in Disjunction 析取取反

翻译忽略， **^** carat就是取反的意思，在disjunction中经常会用到

**^** 只有在析取 **[]** 中作为第一个字符才表示取反，也就是说**^** 与 **[]** 常常连用

| Pattern | Matcher                                        | Examples            |
| ------- | ---------------------------------------------- | ------------------- |
| [^A-Z]  | Not an upper case letter                       | H**ello** W**orld** |
| [^Ss]   | Neither **S** nor **s**                        | S**ay**             |
| [^e^]   | Neither e nor ^                                | ^e**ergg**          |
| a^b     | The pattern a carat b,as a signal tobe matched | **a^b**             |

## More Disjunction 取或

用 **|**来表示，表示取或的意思，或者理解成linux的管道？在前面的基础上继续进行运算。

| Pattern            | Matches                        |
| ------------------ | ------------------------------ |
| hello\|world       | 匹配hello或者world             |
| a\|b\|c            | 等同于[abc]                    |
| [Hh]ello\|[Ww]orld | 匹配大小写开头的hello或者world |

值得注意的是 **|** 判断的是符号之前或之后的连续字符是否匹配，而不是只看前面若干个字符匹配情况。

## ？ * +.

| Patter  | Matches                                      | Instances           |
| ------- | -------------------------------------------- | ------------------- |
| colou?r | optional previous char,?号前面的字符可有可无 | color colour        |
| oo*h!   | 0 or more of previous char,0次或者更多次     | oh!,ooh!,oooh!      |
| o+h!    | 1 or more of previous char,1次或者更多次     | oh!,ooh!            |
| baa+    |                                              | baa,baaa            |
| beg.n   | 1 char,单个字符，可以是任意字符              | beggn，beg1n，beg2n |

## Anchors ^$ 锚

说白了，就是表明位置。**^** 后的字符表示该字符在序列开始处，**$** 前的字符表示该字符在结尾处。

| Pattern     | Matches                                                      |
| ----------- | ------------------------------------------------------------ |
| ^[A-Z]      | **P**alo Alto 只匹配前面的一个大写P                          |
| ^[\^A-Za-z] | **1    "**Hello"  实验证明只能匹配一个字符并不能匹配引号，ppt上有误 |
| \\.$        | The end. 以.结尾                                             |
| .$          | 匹配结尾的任意一个字符 The end**？**  The end**！**          |

## 

# 文本序列化

## Text Normalization 文本规范化

引出几个概念

**lemma** 词根：cat 和 cats的词根是相同的

**wordform** 词形：cat 和 cats的词形式不同的

对于词的类型和数目也有讲究，比如一句简单的英语

they lay back on the San Fancisco grass and looked at the stars and their

关于词的数目，San Fancisco是算一个词还是两个词呢

关于词的类型，they和their算不算同一类词呢？

都是问题

我们通产用N表示token的数目，用V表示词典词的大小，也就是词类型数目

不同的语言在token的过程中会遇到不同的问题，这里就不一一例举了。

课程中还专门介绍了中文分词方法，最大正向匹配法。最大正向匹配法的思想很简单，就是拿词典中最长的词去遍历语句，匹配成功则分词，不成功则换长度更短的词继续匹配，直至匹配结束。

下面介绍集中常见的文本规范化方法。

## Word Normalization

### Case folding

大小写转换，文本处理中我们通常会将大写转化为小写，但是这也会带来一些问题。例如机器翻译中，大写US和小写us完全不是一个意思。但是在信息检索中，我们习惯小写，如果出现了大写也会同时检索小写的结果。

### Lemmatization

词性还原，根据词典找到词初始的词形式，比如单复数，所有格，时态的还原

am，are，is --> be

### Morphology

形态还原，将词分为词干和词缀，在信息检索中就常会用到Stemming。保留词的词干用来检索，

例如compressed和compression都能通过它们的词干compress检索到。

英语中会有专门的规则用来获取词的词干，比如下面这条。

（\*v*)ing  去掉ing后的词干，词干中含有元音

walking  -->walk 	a是元音

sing  -->sing 	 s不是元音，所以词干不是s

秀一波正则表达式，.\*[aoeiu].*ing$，就是上面我们说的匹配所有类似于walking这样的词

## Sentence Segmentation and Decision Trees

句子的划分有时候也是一个问题。英语中的句号.有时会被当作其他用途，并不是真正表示一句话结束，例如%0.2中的.就不是表示句子结束,所以我们要建立一个分类器来判断哪里是句子的开始哪里句子结束了.

例如决策树来判断句子是否结束.

![1bw3v9.png](https://s2.ax1x.com/2020/02/12/1bw3v9.png)

我们需要手工选取许多特征输入决策树让决策树判断

