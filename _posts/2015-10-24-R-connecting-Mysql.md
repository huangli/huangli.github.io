---
layout: post
title:  "R连接Mysql及其乱码问题"
date:   2015-10-24 15:34:37
tags: R
categories: 笔记
---
{% include ext/toc %}


# 概述
其实这篇[博客](http://www.r-bloggers.com/accessing-mysql-through-r/)已经讲的很清楚了关于如何连接Mysql，我的算是存档，另外中文的话需要解决乱码问题。以下代码就解决了连接以及乱码问题


# 连接Mysql

{% highlight R %}
install.packages("RMySQL")
library(RMySQL)
library(
mydb = dbConnect(MySQL(), user='user', password='pwd', dbname='db', host='ip')
dbSendQuery(mydb,'SET NAMES gbk')  
rs = dbSendQuery(mydb, "you sql")
data = fetch(rs, n=-1)
{% endhighlight %}

# 乱码
```dbSendQuery(mydb,'SET NAMES gbk')  ```解决了乱码问题，比较奇怪的是如果用utf8编码本机head或者View函数查看没有问题，但是fix函数查看有问题，而且用gsub函数也有问题。原来Mysql数据库是utf-8的编码。所以我的中文windows 7家庭版智能设置编码为gbk。

# 结果集为空
另外还有一点如果用较为复杂的sql发给Mysql比如我用了Mysql的SUBSTRING和locate函数的话，返回的结果集就是空。所以后来只能不做任何处理，数据清洗放在R中做。


