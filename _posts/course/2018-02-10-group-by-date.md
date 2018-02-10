---
layout: post
title: "如何快速按照时间分组"
author: 大猫
subtitle: "R作为交互性语言的真正的强大之处"
catalog: true
tags:
    - R
    - data.table
---

## 问题的提出
在处理数据的时候，我们常常需要按照日期对数据进行分类汇总，例如每周、每月、每年汇总等。**常见的做法是建立一个用于分类的变量，然后再按照这个变量进行汇总。**然而这种做法特别麻烦，因为我们常常要尝试多种不同的分类长度，很难**事先**就一次性创建好用于分类的变量。

再次，这种常规方法很难处理一些不规则的日期间隔，例如我希望**每隔3天**对数据汇总一次；或者再变态一点，我希望把数据分成两组：一组是周三，另一组是非周三。遇到这种情况，我们该怎么办呢？

本期大猫将教大家使用`data.table`包的`keyby`语句完成上述任务。使用`data.table`的好处是：

- **不需要事先创建分类变量，啥时想分类了，直接分就可以（group on the fly）**
<br>
- **速度特别、特别快！**
<br>
- **代码非常、非常简洁！（也就十几个字符！）**

<br>
## 实战操作
### 生成样例数据集
首先我们生成一个样例数据集：

```{r}
# 生成 100 个日期，从2018-01-01开始
set.seed(42)
n <- 100
dt <- data.table(date = seq(ymd("2018-01-01"), length.out = n, by = "day"), x = runif(n))
```

生成的数据集长这个样子：
![Alt text](/img/in-post/1518117505935.png)

### 按照周进行分类
如果我们想要**每周对变量`x`求均值**，只要在`keyby`语句中指定`week = week(date)`即可：

``` {r}
# 按照周进行分组
dt[, .(x = mean(x)), keyby = .(week = week(date))]
```
结果如下图：
![Alt text](/img/in-post/1518161437807.png)

### 按照星期进行分类
如果想要按照星期（周一到周日）分类，只要把`week`函数改成`wday`即可：
``` {r}
# 按照星期进行分组
res <- dt[, .(x = mean(x)), keyby = .(weekday = wday(date))]
```
结果如下图：
![Alt text](/img/in-post/1518161517710.png)

### 按照“是否为周三”进行分类
如果我们想把样本分成两组，一组是周三（True），一组是非周三（False），则只要使用`wday(date) == 3`来生成一列值为`True`或者`False`的向量就行。
``` {r}
# 按照是否为“周三”进行分组：“True”即周三，“False”即除周三以外的任何日期
dt[, .(x = mean(x)), keyby = .(is.wed = wday(date) == 3)]
```
结果如下：
![Alt text](/img/in-post/1518161707766.png)

### 按照“每个三天”分类
为了按照任意间隔进行分类，我们需要用到`data.table`包中的`ceiling_date`函数。在下面的代码中，`ceiling_date(date, "3 days")`含义是每三天对`date`进行一次“四舍五入”。
``` {r}
# 按照“每3天”进行分组
dt[, .(x = mean(x)), keyby = .(three.day = ceiling_date(date, "3 days"))]
```
大家注意观察最后的结果，是不是每个三天才产生一个输出？
![Alt text](/img/in-post/1518305662962.png)

（完）

