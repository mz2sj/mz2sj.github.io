---
title: 01-二手车价格销售价格预测之EDA
date: 2020-03-24 16:59:22
tags: 数据分析 EDA 回归 比赛
categories: 数据分析 Regression
---



### 写在前面

这次参加了DataWhale组织的二手车销售价值预测组队学习，开始第一部分的学习——数据分析EDA。主要从数据分析流程和软件包工具两方面来介绍自己的收获。

### 数据分析流程

总结了一下，EDA主要做的工作就是发现数据缺失值和查看数据分布情况以及预测目标与其他因变量的相关性分析。

#### 缺失值分析

对于表格数据，使用DataFrame的``info()``函数就可以直观的看到数据的缺失情况，但是这个只针对数值型（numeric）数据，缺失数据会以null值的形式呈现，直接调用DataFrame的`isnull()`函数配合`sum()`就可以对缺失值数据进行统计。比较坑的是object类型数据，有些缺失值数据并不会用null值表示，像下面这种：

![8LKOZ8.png](https://s1.ax1x.com/2020/03/24/8LKOZ8.png)

“-”明显代表缺失值数据，但被字符串代替了。如何发现这种缺失情况，需要我们对相应的数据调用`value_counts()`函数，或者更直观点观察看一部分数据看有无异常情况。比如类别数据中，有的类别只出现了一次，和其他类别压根不是一个数量级，那么很有可能是异常值，可以考虑将其删去。

![8LYuqA.png](https://s1.ax1x.com/2020/03/24/8LYuqA.png)

####  数据分布

首先是预测值的数据分布情况，其次是其他数据（或者说是自变量）的分布情况。对于数值型数据我们可以用正太分布、对数正态分布进行拟合，其他分布自己还不太了解。对于类别特征，我们要查看各种类别对应的数量，可以用柱状图来可视化。也可以查看不同类别的数据对应的目标预测值（price）的分布情况，可以用来分析的图表有箱型图、小提琴图。箱型图可以帮助了解数据的特征范围，也可用来查看离群点。小提琴图在箱型图的基础上显示了目标数据值的分布情况。

![violin](https://datavizcatalogue.com/ZH/%E6%96%B9%E6%B3%95/images/anatomy/SVG/%E5%B0%8F%E6%8F%90%E7%90%B4%E5%9B%BE.svg)

![箱型图](https://niyanchun.com/usr/uploads/2018/07/1539567941.png)

#### 相关性分析

主要是预测值和自变量之间的相关性分析，还有各个自变量之间的相关性分析。常用散点图来绘制，看数据之间是不是有相关性关系。如果预测值和某个自变量有很强的线性关系，那么毫无疑问这个变量对我们的预测是十分有用的。如果有两个自变量它们之间具有很明显的线性关系，那么我们可以只保留其中一个。相关性分析中常用到的图表有热力图、散点图等等。

### 软件工具技能包

以前只知道用pandas读数据，matplotlib绘图，这次学到了一些新知识，在此记录。

#### missingno

见名知意，这个包是用来处理缺失值的。

```python
missingno.matrix(dataframe)
```

![8LtwOH.png](https://s1.ax1x.com/2020/03/24/8LtwOH.png)

```python
missingno.bar(dataframe)
```

![8LtW6g.png](https://s1.ax1x.com/2020/03/24/8LtW6g.png)

可以直观的让我们看出哪些数据有缺失值

#### pandas

```python
DataFrame.skew()
DataFrame.kurt()
```

查看数值型数据的偏度和峰度

```python
DataFrame.corr() 
```

计算数据的相关性

```python
Series.astype('category')
Series.cat.add_category(list)
```

astype将某个Seiries转成category型数据

add_category为category型数据添加新的类别

```python
pd.melt(dataframe,id_vars,value_vars,var_name,value_name)
```

将列名转为列数据，id_vars是保留的列，value_vars是要转换的列，var_name、value_name对应保留和转换后的列名，盗用一张图表示

![melt](https://img-blog.csdn.net/20180927155514513?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21pbmdrb3Vrb3U=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

#### seaborn

有中文的[文档](https://seaborn.apachecn.org/)，查阅起来十分方便

以前绘图使用的都是matplotlib，今天见识到了seaborn的强大，并且绘制出来的图也十分好看

```python
seaborn.distplot(a, bins=None, hist=True, kde=True, rug=False, fit=None, hist_kws=None, kde_kws=None, rug_kws=None, fit_kws=None, color=None, vertical=False, norm_hist=False, axlabel=None, label=None, ax=None)
```

 灵活绘制单变量观测值分布图 ,fit可以配合scipy使用，查看数据是否符合某种数学分布

```python
seaborn.heatmap(data, vmin=None, vmax=None, cmap=None, center=None, robust=False, annot=None, fmt='.2g', annot_kws=None, linewidths=0, linecolor='white', cbar=True, cbar_kws=None, cbar_ax=None, square=False, xticklabels='auto', yticklabels='auto', mask=None, ax=None, **kwargs)
```

可以将pandas处理的corr数据放进去，绘制出热力图，选择不同的主题就很舒服

![cmap](https://seaborn.apachecn.org/docs/img/e18cb02ed3ad1b91b540951f2912539b.jpg)

```python
seaborn.pairplot(data, hue=None, hue_order=None, palette=None, vars=None, x_vars=None, y_vars=None, kind='scatter', diag_kind='auto', markers=None, height=2.5, aspect=1, dropna=True, plot_kws=None, diag_kws=None, grid_kws=None, size=None)
```

diag_kind选择对角线图的类型，可选 {‘auto’, ‘hist’, ‘kde’} 三种类型

```python
seaborn.regplot(x, y, data=None, x_estimator=None, x_bins=None, x_ci='ci', scatter=True, fit_reg=True, ci=95, n_boot=1000, units=None, order=1, logistic=False, lowess=False, robust=False, logx=False, x_partial=None, y_partial=None, truncate=False, dropna=True, x_jitter=None, y_jitter=None, label=None, color=None, marker='o', scatter_kws=None, line_kws=None, ax=None)
```

注意和之前的不同，由于是两个变量之间的回归，所以x,y值得是变量名，data才是DataFrame数据

```python
seaborn.boxplot(x=None, y=None, hue=None, data=None, order=None, hue_order=None, orient=None, color=None, palette=None, saturation=0.75, width=0.8, dodge=True, fliersize=5, linewidth=None, whis=1.5, notch=False, ax=None, **kwargs)
```

同样x、y是字符串，data是数据

```python
seaborn.violinplot(x=None, y=None, hue=None, data=None, order=None, hue_order=None, bw='scott', cut=2, scale='area', scale_hue=True, gridsize=100, width=0.8, inner='box', split=False, dodge=True, orient=None, linewidth=None, color=None, palette=None, saturation=0.75, ax=None, **kwargs)
```

小提琴图的功能与箱型图类似。 它显示了一个（或多个）分类变量多个属性上的定量数据的分布，从而可以比较这些分布。与箱形图不同，其中所有绘图单元都与实际数据点对应，小提琴图描述了基础数据分布的核密度估计。 

```python
seaborn.FacetGrid(data, row=None, col=None, hue=None, col_wrap=None, sharex=True, sharey=True, height=3, aspect=1, palette=None, row_order=None, col_order=None, hue_order=None, hue_kws=None, dropna=True, legend_out=True, despine=True, margin_titles=False, xlim=None, ylim=None, subplot_kws=None, gridspec_kws=None, size=None)
```

来了难点了，以前的只要代数据就可以，这个网格稍微复杂点

 col_wrap 指定网络的列数，col指定列维度，row指定行维度，hue指定第三个维度。在指定维度上，会绘制对应类别的数据。

之后可以调用FacetGrid对象的map方法。像下面的小栗子

``` python
g = sns.FacetGrid(tips, col="time")
g.map(plt.hist, "tip");
```

![map](https://seaborn.apachecn.org/docs/img/5e335e837566d00501c0389d96625993.jpg)

col对应的维度是time，而time有Lunch和Dinner两类数据，就会分别在两列绘制对应的数句。map方法传入要绘制的图类型函数plt.hist,并传递参数名称。

```python
g = sns.FacetGrid(tips, col="sex", hue="smoker")
g.map(plt.scatter, "total_bill", "tip", alpha=.7)
g.add_legend();
```

这是传入第三维度的例子

![hue](https://seaborn.apachecn.org/docs/img/96cf02373c746bde1b58beb44b2f25c8.jpg)

好啦，这次就都这里了，快到deadline了，上传下我的博客。