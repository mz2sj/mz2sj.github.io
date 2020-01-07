---
title: colab使用
date: 2020-01-07 13:26:04
tags: colab Google Drive
---





最近学习pytorch的使用，自己的电脑没有显卡，一些实验做不了，想起之前本科老师安利的colab，感谢谷歌提供的免费资源。

## 访问谷歌

colab基于谷歌云盘，既然是谷歌的服务，难免就要翻墙了。自己用的[shadowsocks](https://github.com/shadowsocks/shadowsocks-windows/releases)，一年100块左右，大部分时间都挺稳定的。开启小飞机，你就可以看到外面的世界啦！

## 云盘挂载

关于colab的基础使用这里就不介绍了，可以自行百度。在使用colab的过程中，经常需要挂载云盘文件，参考了许多人的方法后，介绍一个无坑版。首先，在colab左侧的代码提示部分选择挂载colab会自动生成云盘挂载代码，如下所示：

[![l6wSsJ.md.png](https://s2.ax1x.com/2020/01/07/l6wSsJ.md.png)](https://imgchr.com/i/l6wSsJ)

```python
from google.colab import drive

drive.mount('/gdrive')
```

运行第一个单元格的代码，第二个是打开相关文件的代码忽略。运行后会给出一个超链接，点击进去验证得到验证码输入即可挂载成功，这时候我们挂载的位置是在$/gdrive$ 目录下。那么如何切换到我们想要的目录呢？再次打开左侧的代码提示部分，进入文件栏，点击 $...$ 进入上层目录，进入$/gdrive$目录，可以看到我们云盘下的目录文件，右击复制文件路径。

![l6BdP0.png](https://s2.ax1x.com/2020/01/07/l6BdP0.png)

再通过魔法函数$%cd 路径$就```%cd filepath``` 就可以切换到指定的路径，此时colab的工作路径就是你指定的路径，你就可以像在本地一样操作colab的文件。操作非当前文件夹下的文件，指定好文件的路径即可。

```python
%cd /gdrive/My Drive/Colab Notebooks/测试用的
```

## 无限容量

colab默认的存储容量是15G，可能放几个大文件就占满了，升级的话又不支持支付宝，而且一年也要100多块。贫穷使我想起了歪门邪道。在淘宝上提供了colab扩容服务，20块就够了，其原理是将你加入一些学校的共享盘群组，你可以在上面无限制上传文件。可能会存在一些安全问题，但平时跑跑程序是完全没问题的。加入贡献盘后，我们可以看到自己的gdrive文件夹下多了一个Shared drives。我们可以在上面进行和原硬盘相同的操作，并且无线空间。厉害啦👍

![l6rPXD.png](https://s2.ax1x.com/2020/01/07/l6rPXD.png)

## 纪念自己的第一篇博客

既然开了这个博客就要好好写哦，记录自己的学习、生活，努力成为更好的人啊！

加油

冲