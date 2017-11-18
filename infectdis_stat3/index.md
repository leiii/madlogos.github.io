# 甲乙丙类每月发病、死亡数

```r
library(data.table)
```

看一下甲乙丙类每个月的发病和死亡例数。

```r
sta <- dcast(dat, 日期 ~ 分类, sum, value.var="发病数")
sta <- melt(sta[,names(sta) != "NA"], id="日期", variable.name="分类")
makeTsPlot(sta, "法定传染病每月发病数", xlab="年月", ylab="例数")
```

![](http://ohghnje4x.bkt.clouddn.com/image/170415/inc_trend.png)

```r
sta <- dcast(dat, 日期 ~ 分类, sum, value.var="死亡数")
sta <- melt(sta[,names(sta) != "NA"], id="日期", variable.name="分类")
makeTsPlot(sta, "法定传染病每月死亡数", xlab="年月", ylab="例数")
```

![](http://ohghnje4x.bkt.clouddn.com/image/170415/mot_trend.png)

甲类数字很少，看不太出。而不论乙类还是丙类，发病高峰都在春夏季。死亡高峰却在冬季。

按月算一下均数，看得更清楚。

```r
sta <- dcast(dat, format(日期, "%m") ~ 分类, mean, value.var="发病数")
names(sta)[1] <- "月份"
sta <- melt(sta[,1:4], id="月份", variable.name="分类")
sta$月份 <- as.integer(sta$月份)
makeTsPlot(sta, "法定传染病平均月发病数", unit=1, ylab="平均例数", xvar="月份")
```

![](http://ohghnje4x.bkt.clouddn.com/image/170415/inc_month.png)

```r
sta <- dcast(dat, format(日期, "%m") ~ 分类, mean, value.var="死亡数")
names(sta)[1] <- "月份"
sta <- melt(sta[,1:4], id="月份", variable.name="分类")
sta$月份 <- as.integer(sta$月份)
makeTsPlot(sta, "法定传染病平均月死亡数", unit=1, ylab="平均例数", xvar="月份")
```

![](http://ohghnje4x.bkt.clouddn.com/image/170415/mot_month.png)

<!--more-->

# 乙类

## 四大类别

把乙类归成肠道、呼吸道、血源/性、虫媒/自然疫源地四大类。

```r
dat.b <- subset(dat, 分类=="乙类" | str_detect(病名, "肝炎"))
dat.b <- dat.b[dat.b$病名 != "病毒性肝炎",]
dat.b$类型 <- NA
dat.b$类型[str_detect(
    dat.b$病名, "[甲戊]型肝炎|痢疾|伤寒|脊髓灰质炎")] <- "肠道"
dat.b$类型[str_detect(
    dat.b$病名, "结核|麻疹|猩红热|流感|百日咳|脑脊髓膜炎|禽流感|白喉|肺炎")] <- "呼吸道"
dat.b$类型[str_detect(
    dat.b$病名, "布鲁氏|疟疾|出血热|血吸虫|登革|乙型脑炎|狂犬|钩端螺旋体|炭疽")] <- "虫媒/自然疫源"
dat.b$类型[str_detect(
    dat.b$病名, "[乙丙丁]型肝炎|梅毒|淋病|艾滋病|破伤风|肝炎未分型")] <- "血源/性传"
```

一个明显趋势是血源/性传播疾病占比越来越高。这个趋势在2008-2010年左右已经很明显，至今没有减退，从死亡数占比来看，现在更上了一个台阶。几乎要垄断行情了。

```r
sta <- dcast(dat.b, 日期 ~ 类型, sum, value.var="发病数")
sta <- melt(sta, id="日期", variable.name="类型")
makeTsPlot(sta, "乙类传染病每月发病数", xlab="年月", ylab="例数", gvar="类型")
```

![](http://ohghnje4x.bkt.clouddn.com/image/170415/inc_b_trend.png)

```r
sta <- dcast(dat.b, 日期 ~ 类型, sum, value.var="死亡数")
sta <- melt(sta, id="日期", variable.name="类型")
makeTsPlot(sta, "乙类传染病每月死亡数", xlab="年月", ylab="例数", gvar="类型")
```

![](http://ohghnje4x.bkt.clouddn.com/image/170415/mot_b_trend.png)

## 详细病种

究竟是哪个具体病种发展更快？

```r
sta <- dcast(dat.b, 病名~., sum, value.var="发病数")
top.b <- sta[order(sta$., decreasing=TRUE), "病名"][1:10]
sta <- dcast(dat.b, 日期 ~ 病名, sum, value.var="发病数")
sta <- melt(sta, id="日期", variable.name="病名")
sta$病名 <- as.character(sta$病名)
sta$病名[! sta$病名 %in% top.b] <- "其它"
sta <- dcast(sta, 日期 + 病名~., sum, value.var="value")
sta$病名 <- factor(sta$病名, levels=c(top.b, "其它"))
makeTsPlot(sta, "乙类传染病每月发病数", xlab="年月", ylab="例数", yvar=".",
           gvar="病名", legend.position = "bottom")
```

![](http://ohghnje4x.bkt.clouddn.com/image/170415/inc_b_det_trend.png)

```r
sta <- dcast(dat.b, 病名~., sum, value.var="死亡数")
top.b <- sta[order(sta$., decreasing=TRUE), "病名"][1:10]
sta <- dcast(dat.b, 日期 ~ 病名, sum, value.var="死亡数")
sta <- melt(sta, id="日期", variable.name="病名")
sta$病名 <- as.character(sta$病名)
sta$病名[! sta$病名 %in% top.b] <- "其它"
sta <- dcast(sta, 日期 + 病名~., sum, value.var="value")
sta$病名 <- factor(sta$病名, levels=c(top.b, "其它"))
makeTsPlot(sta, "乙类传染病每月死亡数", xlab="年月", ylab="例数", yvar=".",
           gvar="病名", legend.position = "bottom")
```

![](http://ohghnje4x.bkt.clouddn.com/image/170415/mot_b_det_trend.png)

> 乙类死亡数分布中，2009年末-2010年初有个醒目的浅蓝色楔子。那就是著名的甲型H1N1流感流行。

从发病数看，梅毒越来越多了，夏季高发。丙肝也越来越多了，冬春季高发。

从死亡数看，艾滋病单一病种吃掉了越来越大的份额。

说到底，传染病控制的重心基本上不可逆转地会朝这几个方向移动。



## 肝炎

肝炎是细分报告的。所以也可以下钻看一眼。

先析出一个分型肝炎子集。

```r
dat.hep <- subset(dat, str_detect(病名, "^肝炎|[^性]肝炎"))
dat.hep$病名 <- str_replace(dat.hep$病名, "([甲乙丙丁戊])型肝炎|^肝炎(未分)型", "\\1\\2")
dat.hep$病名 <- factor(dat.hep$病名, levels=c("甲", "乙", "丙", "丁", "戊", "未分型"))
```

然后分别看发病和死亡。

```r
sta <- dcast(dat.hep, 日期 ~ 病名, sum, value.var="发病数")
sta <- melt(sta, id="日期", variable.name="型别")
makeTsPlot(sta, "肝炎每月发病数", xlab="年月", ylab="例数", gvar="型别")
```

![](http://ohghnje4x.bkt.clouddn.com/image/170415/inc_hep_trend.png)

```r
sta <- dcast(dat.hep, 日期 ~ 病名, sum, value.var="死亡数")
sta <- melt(sta, id="日期", variable.name="型别")
makeTsPlot(sta, "肝炎每月死亡数", xlab="年月", ylab="例数", gvar="型别")
```

![](http://ohghnje4x.bkt.clouddn.com/image/170415/mot_hep_trend.png)

感觉都在慢慢下降。

# 丙类

析出一个子集。

```r
dat.c <- subset(dat, 分类=="丙类" & 日期 >= as.Date("2009-1-1"))
```

## 不同病种的时间趋势

```r
sta <- dcast(dat.c, 日期 ~ 病名, sum, value.var="发病数")
sta <- melt(sta, id="日期", variable.name="病名")
makeTsPlot(sta, "丙类传染病每月发病数", xlab="年月", ylab="例数", gvar="病名",
           legend.position = "bottom")
```

![](http://ohghnje4x.bkt.clouddn.com/image/170415/inc_c_det_trend.png)

```r
sta <- dcast(dat.c, 日期 ~ 病名, sum, value.var="死亡数")
sta <- melt(sta, id="日期", variable.name="病名")
makeTsPlot(sta, "丙类传染病每月死亡数", xlab="年月", ylab="例数", gvar="病名",
           legend.position = "bottom")
```

![](http://ohghnje4x.bkt.clouddn.com/image/170415/mot_c_det_trend.png)

其实就两样：手足口、感染性腹泻。落到死亡，基本都是手足口。

丙类传染病占据了基层疾控主要的流调精力，但其实能死人的也就是手足口。

## 各病种的平均月分布

```r
sta <- dcast(dat.c, format(日期, "%m") ~ 病名, mean, value.var="发病数")
names(sta)[1] <- "月份"
sta <- melt(sta, id="月份", variable.name="病名")
sta$月份 <- as.integer(sta$月份)
makeTsPlot(sta, "丙类传染病平均月发病数", unit=1, ylab="平均例数", xvar="月份",
           gvar="病名", legend.position = "bottom")
```

![](http://ohghnje4x.bkt.clouddn.com/image/170415/inc_c_month.png)

```r
sta <- dcast(dat.c, format(日期, "%m") ~ 病名, mean, value.var="死亡数")
names(sta)[1] <- "月份"
sta <- melt(sta, id="月份", variable.name="病名")
sta$月份 <- as.integer(sta$月份)
makeTsPlot(sta, "丙类传染病平均月死亡数", unit=1, ylab="平均例数", xvar="月份",
           gvar="病名", legend.position = "bottom")
```

![](http://ohghnje4x.bkt.clouddn.com/image/170415/mot_c_month.png)

看月份分布，春夏季是大头。

## 流感

额外关心了一下流感。

```r
dat.flu <- subset(dat, 病名 =="流行性感冒" & 日期 >= as.Date("2009-1-1"))
makeTsPlot(dat.flu, "流感每月发病数", xlab="年月", ylab="例数", gvar="病名",
           xvar="日期", yvar="发病数")
```

![](http://ohghnje4x.bkt.clouddn.com/image/170415/inc_flu_trend.png)

2016年初春有一个高峰。今明两年估计不会有那么高了。

# 结尾

上面这些是很粗浅的分析。用shiny结合这些数据做一个仪表盘是再合适不过的了。配点时间序列模型和预测，整个仪表盘就很丰富实用了。可惜印象中并没有这类公共的数据产品出来。可能也有，但多半藏在某些衙门的某些电脑上离线运行着。

离开疾控至今，还没有再关注过传染病的动态。当初上课时，老师还提到“死亡数最多的传染病你们或许猜不到，是狂犬病”。后来变成了结核。如今，已完全是艾滋病的天下了。短短几年，这个静默无闻的领域也发生着剧变。

[完]

----

<img src="http://ohghnje4x.bkt.clouddn.com/QRcode.jpg" width="50%">扫码关注我的的我的公众号</img>

