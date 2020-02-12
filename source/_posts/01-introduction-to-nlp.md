---
title: 01 introduction to nlp
date: 2020-02-12 16:10:18
tags: nlp
categories: nlp
---

## 写在前面

研究生读了半年了，越来越迷茫了，没有认认真真做好一件事。今天，立帖为证，我一定要将这门课认真修完，勤做笔记，完成作业。话不多说，开始吧！附上[课程链接]( https://www.bilibili.com/video/av35805262 )。

## nlp的应用领域

1. Question Answering 问答

2. Information Extraction 信息抽取

3. Sentiment Analysis 情感分析

4. 机器翻译

自然语言在垃圾邮件侦测、词性标注、命名实体识别上已经取得了较好的表现，在情感分类、指代消解、句法分析、机器翻译、信息抽取也有所进步，尤其是现在的神经机器翻译，比之前的统计机器翻译取得了长足的进步，不过这门课开设时间已经比较早了，那时候深度学习还没有大规模应用到nlp中。但在问答、摘要、对话系统、改述（Paraphrase）等方面表现还有待提高。

## 歧义让nlp变得更加困难

举个例子，这是纽约时报的一条新闻头条

Fed raises interest rates

既可以理解为Fed这个人将利率interest rates 提高了，也可以理解为Fed这个人对rates这个东西起了兴趣raise interest。

当给出更多信息时，可以帮助我们消除这种模糊性。

Fed raises interest rates 0.5%

这时候我们就知道了新闻真实想表达的意思是利率提高了%0.5，而不是Fed来了兴趣.

## 其他的困难

1. 非标准英语 比如推特上的用语可能就很不规范
2. 切分问题 一个词不同的切分会带来不同的问题
3. 俚语 类似于我们的成语,不能通过字面意思来判断他的意思
4. 新词 类似于给力这种新词汇
5. 外部知识 知识的不确定性
6. 实体名 有些词可能是专有名词,不能按字面意思理解

## 写在后面

第一篇差不多就到这儿了,完事开头难,加油写下去哦!