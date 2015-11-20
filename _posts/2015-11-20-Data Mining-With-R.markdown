---
layout: post
title:  "R探索数据分析(持续更新)"
date:   2015-11-20 15:34:37
tags: 读书笔记 R
categories: jekyll update
---

## 概述
用来探索数据的工具不少，比如不需要任何编程经验的[Tableau](http://www.tableau.com/)、Excel等，
此文是总结如何用R来完成探索数据或者描述性统计分析的工作，虽然R灵活但是学习曲线的确比Tableau高了一些。数据和代码来自与[Data Mining with R](http://www.dcc.fc.up.pt/~ltorgo/DataMiningWithR/datasets4.html)的sales和algae数据

### sales 数据
此文所有操作都是针对sales数据。原始数据做个简单介绍：

- ID: 销售人员编号
- Prod：产品编码编号
- Quant: 产品销售数量
- Val: 销售金额
- Insp: fruad代表是欺诈，ok没有，unkn不知道
- Uprice: 自己算的单价，金额/数量

### algae
algae就是关于河流的各种化学参数了，不做详细解释。

## 数据探索技巧
### Top N/Bottom N 

如何找到sales数据中产品价格中位数最贵和最便宜的5个产品？我的思路是先求出产品价格中位数，然后找到最贵的、接着最便宜的最后两个数据和在一起。作者代码显然更精炼。
通过order函数进行排序，sapply函数把最贵和最便宜和在一起，接着通过boxplot的方式看分布,这里用了指数形式否则差别太大有些数据不易观察到。
{% highlight R %}
attach(sales)
upp <- aggregate(Uprice, list(Prod), median, na.rm=T)    #相当于group by
topP <- sapply(c(T,F), function(o) upp[order(upp[,2], decreasing=o)[1:5],1])
colnames(topP) <- c("Expensive", "Cheap")

# compare the most expensive and cheap product
tops <- sales[Prod %in% topP[1,], c("Prod", "Uprice")]
tops$Prod <- factor(tops$Prod)
boxplot(Uprice ~ Prod, data = tops, ylab="Uprice", log="y")
{% endhighlight %}


### 找到异常值outlier

如何找到sales数据产品价格的异常值并统计数量？如果人群之中姚明站在那里，一眼就能看出来。那么R有什么办法可以找到异常值？建议先看看什么是[四分位数](https://zh.wikipedia.org/wiki/%E5%9B%9B%E5%88%86%E4%BD%8D%E6%95%B0)和[箱形图](http://stattrek.com/statistics/charts/boxplot.aspx?Tutorial=AP)，便可以了解判断的[方法](http://www.itl.nist.gov/div898/handbook/prc/section1/prc16.htm)。R有方法[boxplot.stats](https://stat.ethz.ch/R-manual/R-devel/library/grDevices/html/boxplot.stats.html)。
{% highlight R %}
attach(sales)
out <- tapply(Uprice,list(Prod=Prod), function(x) length(boxplot.stats(x)$out))
out[order(out, decreasing = T)[1:10]] #查看前10 
sum(out)   #得到所有outlier的数量
sum(out)/nrow(sales) * 100 #得到百分比
{% endhighlight %}

## 参考资料
- [四分位数](https://zh.wikipedia.org/wiki/%E5%9B%9B%E5%88%86%E4%BD%8D%E6%95%B0)
- [箱形图](http://stattrek.com/statistics/charts/boxplot.aspx?Tutorial=AP)
- [outlier](http://www.itl.nist.gov/div898/handbook/prc/section1/prc16.htm)


