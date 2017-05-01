---
layout: post
title: 【动态图版】北宋、明清进士空间可视化分析
description: "利用animiation、ECharts2和leaflet将进士地理信息动态可视化到地图上。"
author: madlogos
modified: 2017-4-30
tags: [可视化]
comments: true
---

> 如果浏览器提示本文加载了不安全的脚本，请点允许。

今次主题比较简单。上个话题留了点冷饭，看起来还没馊，咱敲个鸡蛋炒个蛋炒饭。

炒个什么蛋炒饭呢？——动态图(dynamic charts)。这也是应留言要求额外发的番外。基本和分析关系不大，纯粹是可视化范畴。

# 动态图

通常我们看到的都是静态图，最常见的是.jpg、.png这类位图，逼格高一点的会用到.svg矢量图。但它们都是死图，所有图形元素都不会动。某些情况下，我们不仅要把统计结果映射到特定的视觉通道，还希望表现其历时性，或者允许用户自己进行挖掘。这就需要让图形部件动起来。

- 一种方法是做成动画(animation)。

R有个名包，叫animation，可以用它压制.gif，用在社交媒体效果足够醒目。它的基本思路就是拿出一维来映射时间，基于时间点对数据切片、统计、制图，最后把静态图们合成一个动画。

- 还有一种方法是交互图(interactive charts)。

把统计数据绑定到JavaScript控件上，定义好交互方法，用户即可在网页上通过控件操作来调整视觉呈现（切片、缩放、改变类型等）。RStudio发过一个工具框架包htmlwidgets，可以很方便地把已有的JavaScript可视化库移植到R。我们今天就要用到其中的两个：ECharts2和leaflet。

如果再进一步，就是数据交互面板了。R有shiny及其系列衍生品，比如flexboard。想象一下作战室交互图仪表盘面板，几行命令就做出来了。简直酷炫。但是这需要部署在shiny服务器上。

# 动画

还是用前次的数据集，再画一下各朝的进士地理热图。但这次变个花样，我们先把北宋、明、清都按前、中、后分一下期。

- 用cut函数把连续变量切分成分段因子；
- 用sprintf格式化因子标签，否则会自动套用前开后闭科学计数法区间样式，表示年代比较怪；
- 各朝的分期不见得都对，我就随便分了下：
	- 北宋(960-1127)：以1021、1085分期，大体对应真宗/仁宗、神宗/哲宗；
	- 明朝(1668-1644)：以1434、1572分期，大体对应宣宗(宣德)/英宗、穆宗(隆庆)/神宗(万历)；
	- 清朝(1644-1911)：以1735、1850分期，大体对应世宗(雍正)/高宗(乾隆)、宣宗(道光)/文宗(咸丰)。

```r
library(animation)
nsong.js$period <- cut(
    nsong.js$EntryYear, c(960, 1021, 1085, 1127), include.lowest=TRUE,
    labels=paste(sprintf("%4d", cutyears[1:(length(cutyears)-1)]),
                 sprintf("%4d", cutyears[2:length(cutyears)]), sep="-"))
ming.js$period <- cut(
    ming.js$EntryYear, c(1368, 1434, 1572, 1644), include.lowest=TRUE,
    labels=paste(sprintf("%4d", cutyears[1:(length(cutyears)-1)]),
                 sprintf("%4d", cutyears[2:length(cutyears)]), sep="-"))
qing.js$period <- cut(
    qing.js$EntryYear, c(1644, 1735, 1850, 1911), include.lowest=TRUE,
    labels=paste(sprintf("%4d", cutyears[1:(length(cutyears)-1)]),
                 sprintf("%4d", cutyears[2:length(cutyears)]), sep="-"))
```

各自切片统计，再做成.gif动画。这里设置动画间隔为2秒，每一帧大小都是640×480像素。

```r
oopt <- ani.options(interval=2, ani.width=640, ani.height=480)
saveGIF({
    for (i in levels(nsong.js$period)){
        print(p.song + stat_density_2d(aes(x_coord, y_coord, fill=..level..), 
        data=nsong.js[nsong.js$period==i,], geom="polygon", alpha=0.5) +
        scale_fill_gradient(low="cyan", high="darkblue")+
        ggtitle(paste0("北宋进士来源地 (", i, ")")))
    }
}, "song.gif")
```

![](http://ohghnje4x.bkt.clouddn.com/image/170430/song.gif)

很明显，前期京畿还很有优势，仁宗开始江南大盛，到神宗以后就是闽人大盛。热力分布持续东南移。

```r
saveGIF({
    for (i in levels(ming.js$period)){
        print(p.ming + stat_density_2d(aes(x_coord, y_coord, fill=..level..), 
        data=ming.js[ming.js$period==i,], geom="polygon", alpha=0.5) +
        scale_fill_gradient(low="cyan", high="darkblue")+
        ggtitle(paste0("明朝进士来源地 (", i, ")")))
    }
}, "ming.gif")
```

![](http://ohghnje4x.bkt.clouddn.com/image/170430/ming.gif)

明和北宋热力分布是反向变动的，神宗以后进士来源地更弥散了。赣闽次第没落，而京畿有了起色。

```r
saveGIF({
    for (i in levels(qing.js$period)){
        print(p.qing + stat_density_2d(aes(x_coord, y_coord, fill=..level..), 
        data=qing.js[qing.js$period==i,], geom="polygon", alpha=0.5) +
        scale_fill_gradient(low="cyan", high="darkblue")+
        ggtitle(paste0("清朝进士来源地 (", i, ")")))
    }
}, "qing.gif")
ani.options(oopt)
```

![](http://ohghnje4x.bkt.clouddn.com/image/170430/qing.gif)

清和明类似，中前期江南独大，鸦片战争以后分布逐渐弥散。

和静态图相比，引入历时维度后，信息量明显就更大了。

# 交互图

## 个体散点图

在哈佛人文地理平台上，进士数据集本身就以散点和热力图形式展示。这有一个问题：同一个来源地的坐标是完全一样的，所以这些个体在空间上都叠在一起，无法区分。在用leaflet再现散点分布之前，我们用随机数把他们打散开。

```r
nsong.js$long <- nsong.js$x_coord + rnorm(nrow(nsong.js))/100
nsong.js$lat <- nsong.js$y_coord + rnorm(nrow(nsong.js))/100
ming.js$long <- ming.js$x_coord + rnorm(nrow(ming.js))/100
ming.js$lat <- ming.js$y_coord + rnorm(nrow(ming.js))/100
qing.js$long <- qing.js$x_coord + rnorm(nrow(qing.js))/100
qing.js$lat <- qing.js$y_coord + rnorm(nrow(qing.js))/100
```

这样一来，低级缩放时这些点还是叠在一起的，只要持续放大，就能散开。这当然很不精确，但是我感觉能忍。

构造一个工作函数`make_leaflet`，把参数传进去，就能画个leaflet交互图出来。leaflet的语法很简单，就是个leaflet js库的wrapper(咋翻译？包裹函数？)，参数列表几乎都是从API文档继承来的。基本的使用顺序是初始化->添加地图图层->添加主题图层->添加交互控件。我没多玩花活，加了个label和popup。

leaflet是基于svg矢量图的，源代码里就包含很多矢量代码。加上这里数据点多，文件尺寸就都很大。

```r
library(leaflet)
make_leaflet <- function(refMap, dyn, bgColor="red", dataset, cutyears){
    dataset$period <- cut(
        dataset$EntryYear, cutyears, include.lowest=TRUE,
        labels=paste(sprintf("%4d", cutyears[1:(length(cutyears)-1)]),
                     sprintf("%4d", cutyears[2:length(cutyears)]), sep="-"))
    dataset <- dataset[order(dataset$period),]
    g <- leaflet() %>% addProviderTiles(providers$OpenStreetMap)
    for (i in unique(refMap$id)){
        g <- g %>% 
            addPolygons(~long, ~lat, data=refMap[refMap$id==i,], 
		        label=dyn, labelOptions=labelOptions(textsize="20px"),
		        weight=1, fillColor=bgColor, color=bgColor)
    }
    pal <- colorFactor(
        viridis::plasma(2*nlevels(dataset$period)), 
        levels=levels(dataset$period), ordered=TRUE)
    g <- g %>% addCircleMarkers(
        ~long, ~lat, data=dataset, radius=2, 
        label=paste(dataset$NameChn, dataset$Name),
		popup=paste(dataset$EntryYear, "<br>", dataset$Prov, 
		            dataset$AddrChn, dataset$AddrName, sep=" "),
		color=~pal(period), group=~period)
    g %>% addLayersControl(
        overlayGroups=rev(levels(dataset$period)),
        options=layersControlOptions(
            collapse=FALSE)
        )
}
```

```r
make_leaflet(nsong.bou, "北宋", "red", nsong.js, c(960, 1021, 1085, 1127))
```

<iframe src="http://ohghnje4x.bkt.clouddn.com/html/170430/song.html" width="100%" height="500"></iframe>

[点开查看源文件](http://ohghnje4x.bkt.clouddn.com/html/170430/song.html)


```r
make_leaflet(ming.bou, "明朝", "red", ming.js, c(1368, 1434, 1572, 1644))
```

<iframe src="http://ohghnje4x.bkt.clouddn.com/html/170430/ming.html" width="100%" height="500"></iframe>

[点开查看源文件](http://ohghnje4x.bkt.clouddn.com/html/170430/ming.html)

```r
make_leaflet(qing.bou, "清朝", "black", qing.js, c(1644, 1735, 1850, 1911))
```

<iframe src="http://ohghnje4x.bkt.clouddn.com/html/170430/qing.html" width="100%" height="500"></iframe>

[点开查看源文件](http://ohghnje4x.bkt.clouddn.com/html/170430/qing.html)

## 古今地名一致性

前次分析，直接把所有县级地名拿出来跑频数，第一名是莆田。这当然很不精准——地名一直会变嘛。像南京这种历史上频繁改名的地方，就会吃很大亏。最好统一起来。找地名的历史沿革数据库不是很容易，干脆统一到今名如何？（请都说“好”）

统一到今名有三个好处：

1. 能按照今天的区划统计哪个省市出了更多进士（其实谁稀罕这个）；
2. 便于我操作（-口_口-：仄寺坠重要的）；
3. 能引出一个知识点：sp::over()。

既然我们有每个省/市级空间多边形数据，又有每个进士的位置坐标，恰好又都是同一套大地坐标系测量值，历史沿革问题不就转化成了一个纯粹的GIS转换问题吗？只要用算法评价每个点落在哪个多边形内，它自然就属于这个行政区划。

这个算法有现成的，就是sp包的over函数。

构造一个`which_polygon`函数，把一个点轮流放进大陆和台湾的多边形数据集里，看它到底落在哪个多边形，输出这个多边形的元数据。注意：数据都得是sp包的工作类型，比如SpatialPoints、SpatialPolygons之类。

```r
which_polygon <- function(point, cn.cities, tw.cities){
    library(sp)
    out <- over(SpatialPoints(
        matrix(point, nrow=1), 
        proj4string = cn.cities@proj4string), cn.cities)
    if (is.na(out$ID_0))
        out <- over(SpatialPoints(
            matrix(point, nrow=1), 
            proj4string = tw.cities@proj4string), tw.cities)
    return(out)
}
```

每个进士个体都丢进去算太耗时，我们先统计下频数。

```r
library(data.table)
nsong.js.stat <- dcast(nsong.js, x_coord+y_coord~., length)
ming.js.stat <- dcast(ming.js, x_coord+y_coord~., length)
qing.js.stat <- dcast(qing.js, x_coord+y_coord~., length)
```

再次动用并行计算parallel。因为要按行遍历矩阵，所以要用`apply`的并行版本`parApply`。获得结果是个list，需要再do.call调用bind_rows，合并成一个数据框。当然还有种办法是直接改写which_polygon，把进士位置整理成SpatialPointsDataFrame，一次性丢进去，效率上可能更高。

```r
library(parallel)
cl <- makeCluster(getOption("cl.cores", 2))
nsong.js.belong <- parApply(cl, nsong.js.stat[,c("x_coord", "y_coord")], 
                            1, which_polygon, cn.cities, tw.cities)
ming.js.belong <- parApply(cl, ming.js.stat[,c("x_coord", "y_coord")], 
                           1, which_polygon, cn.cities, tw.cities)
qing.js.belong <- parApply(cl, qing.js.stat[,c("x_coord", "y_coord")], 
                           1, which_polygon, cn.cities, tw.cities)
stopCluster(cl)
library(dplyr)
nsong.js.belong <- do.call('bind_rows', nsong.js.belong)
ming.js.belong <- do.call('bind_rows', ming.js.belong)
qing.js.belong <- do.call('bind_rows', qing.js.belong)
nsong.js.belong$num <- nsong.js.stat$`.`
ming.js.belong$num <- ming.js.stat$`.`
qing.js.belong$num <- qing.js.stat$`.`
```

## 进士数排名

把三个朝代的数据集做一下深加工，合并到一起（DYNASTY列标识朝代属性）。

```r
library(data.table)
nsong.js.belong <- dcast(nsong.js.belong, NAME_1+NAME_2+NL_NAME_2~., sum, value.var="num")
nsong.js.belong$DYNASTY <- "北宋"
ming.js.belong <- dcast(ming.js.belong, NAME_1+NAME_2+NL_NAME_2~., sum, value.var="num")
ming.js.belong$DYNASTY <- "明朝"
qing.js.belong <- dcast(qing.js.belong, NAME_1+NAME_2+NL_NAME_2~., sum, value.var="num")
qing.js.belong$DYNASTY <- "清朝"
library(dplyr)
js.belong <- do.call("bind_rows", list(nsong.js.belong, ming.js.belong, qing.js.belong))
```

再清洗一下数据，把英文地名里的无关符号删掉（“内江”出现了两种写法），再把0都转为NA（可视化时不要涂色）。

```r
js.belong$NAME_2 <- stringr::str_replace(js.belong$NAME_2, "^([^]]+)\\]+$", "\\1")
js.belong$.[js.belong$.==0] <- NA
```

排名来了。

### 先是市级排名

```r
js.order <- dcast(js.belong, NAME_1+NAME_2+NL_NAME_2~DYNASTY, sum, 
                  value.var=".", margins="DYNASTY")
knitr::kable(js.order[order(js.order$`(all)`, decreasing=TRUE),])
```

地名归集后，第一名是苏州，然后是福州，第三才是莆田，第四是杭州。前四名实际难分伯仲。

| 序号   | NAME_1	| NAME_2	| NL_NAME_2	| 北宋	| 明朝	| 清朝	| (all) |
|------:|-----------|-----------|-----------|------:|------:|------:|------:|
|  1	| Jiangsu	| Suzhou	| 苏州市	| 157	| 60	| 179	| 396 |
|  2	| Fujian	| Fuzhou	| 福州市	| 273	| 36	| 83	| 392 |
|  3	| Fujian	| Putian	| 莆田市	| 264	| 109	| 8		| 381 |
|  4	| Zhejiang	| Hangzhou	| 杭州市	| 115	| 38	| 221	| 374 |
|  5	| Fujian	| Nanping	| 南平市	| 224	| 15	| 7		| 246 |
|  6	| Jiangxi	| Ji’an		| 吉安市	| 128	| 97	| 11	| 236 |
|  7	| Zhejiang	| Ningbo	| 宁波市	| 94	| 97	| 39	| 230 |
|  8	| Jiangsu	| Changzhou	| 常州市	| 97	| 20	| 97	| 214 |
|  9	| Zhejiang	| Jiaxing	| 嘉兴市	| 19	| 68	| 117	| 204 |
| 10	| Jiangxi	| Fuzhou	| 抚州市	| 120	| 27	| 52	| 199 |
| 11	| Zhejiang	| Huzhou	| 湖州市	| 86	| 17	| 78	| 181 |
| 12	| Zhejiang	| Shaoxing	| 绍兴市	| 75	| 48	| 51	| 174 |
| 13	| Fujian	| Quanzhou	| 泉州市	| 110	| 28	| 31	| 169 |
| 14	| Jiangsu	| Wuxi		| 无锡市	| 73	| 34	| 55	| 162 |
| 15	| Jiangxi	| Shangrao	| 上饶市	| 93	| 26	| 28	| 147 |

给张图，同样是基于leaflet的。注意：**文件非常很大，极费流量**。

<iframe src="http://ohghnje4x.bkt.clouddn.com/html/170430/city.html" width="100%" height="500"></iframe>

[点开查看源文件](http://ohghnje4x.bkt.clouddn.com/html/170430/city.html)

然后是省级排名。

```r
js.order.prov <- dcast(js.belong, NAME_1~DYNASTY, sum, value.var=".", margins="DYNASTY")
knitr::kable(js.order.prov[order(js.order.prov$`(all)`, decreasing=TRUE),])
```

|序号|NAME_1		|北宋	|明朝	|清朝	|(all)|
|--:|-----------|------:|------:|------:|-----:|
| 1	|Zhejiang	|636	|337	|530	|1503|
| 2	|Fujian		|950	|202	|147	|1299|
| 3	|Jiangsu	|476	|179	|536	|1191|
| 4	|Jiangxi	|559	|233	|232	|1024|
| 5	|Henan		|276	|97		|185	|558|
| 6	|Shandong	|99		|88		|311	|498|
| 7	|Anhui		|142	|68		|197	|407|
| 8	|Sichuan	|269	|30		|65		|364|
| 9	|Hebei		|27		|86		|200	|313|
|10	|Hunan		|79		|27		|152	|258|
|11	|Hubei		|31		|52		|133	|216|
|12	|Shanxi		|24		|63		|125	|212|
|13	|Guangdong	|40		|54		|105	|199|
|14	|Shaanxi	|18		|49		|89		|156|
|15	|Shanghai	|26		|47		|68		|141|
|16	|Beijing	|2		|12		|115	|129|
|17	|Yunnan		|0		|1		|69		|70|
|18	|Guangxi	|2		|10		|56		|68|
|19	|Tianjin	|0		|2		|61		|63|
|20	|Guizhou	|0		|4		|51		|55|
|21	|Chongqing	|8		|7		|24		|39|
|22	|Gansu		|6		|6		|19		|31|
|23	|Hainan		|0		|9		|3		|12|
|24	|Liaoning	|0		|0		|12		|12|
|25	|Jilin		|0		|0		|8		|8|
|26	|Ningxia Hui|0		|0		|7		|7|
|27	|Taiwan		|0		|0		|2		|2|

同样给张图。这次是基于ECharts2的。由于这个包是我写的，所以我要额外安利一下http://madlogos.github.io/recharts。

```r
library(recharts)
dict <- geoNameMap[geoNameMap$FKEY==31, c("EN", "CN")]
js.order.prov <- merge(js.order.prov, dict, by.x="NAME_1", by.y="EN", all.x=TRUE)
js.order.ec <- melt(js.order.prov[,c(2:4,6)], id="CN")
js.order.ec$value[js.order.ec$value==0] <- NA
echartR(js.order.ec, CN, value, t=variable, type="map_china", subtype="average") %>%
    setDataRange(splitNumber=0, color=c('darkblue','cyan')) %>%
    setTitle("各省进士数", pos=11) %>% 
    setTimeline(autoPlay=TRUE) %>% setLegend(FALSE)
```

<iframe src="http://ohghnje4x.bkt.clouddn.com/html/170430/prov.html" width="640" height="500"></iframe>

[点开查看源文件](http://ohghnje4x.bkt.clouddn.com/html/170430/prov.html)

图动起来了。感觉棒呆。

自己试着去和它们交互吧。

[完]

----

<img src="http://ohghnje4x.bkt.clouddn.com/QRcode.jpg" width="50%">扫码关注我的的我的公众号</img>
