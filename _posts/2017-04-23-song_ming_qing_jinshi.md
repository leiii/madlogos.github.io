---
layout: post
title: 北宋、明清进士空间分析
description: "利用哈佛地理可视化平台、中国历代人物传记数据库的标记数据集，以及“发现中国”导出的地理数据，绘制北宋、明、清的进士来源地，以及它们比重的历时变迁。莆田是进士第一来源地，但至清时，福建、江西的比重已经不高。"
author: madlogos
modified: 2017-4-23
tags: [数据]
comments: true
---

# 哈佛人文地理可视化平台

周末找在线GIS素材，误打误撞进到了worldmap.harvard.edu——一座名副其实的学术宝库。它是哈佛大学搞的一个**开源**地理可视化平台，社会、经济、历史学科的学者可以自己创建、上传数据集，方便地做多图层地理可视化。这个设计充满学术理想，也为有志于成为世界一流名校的大学树立了一座看得见摸得着的价值标杆：**卓越、开放、自治、共享**。

中国地图worldmap.harvard.edu/chinamap是其中一个子站，已经上线多个数据层，包括社会/人口、经济、交通、能源、环境/气候、公共卫生，甚至历史地图。比如下面的视图，就是细化到县级的人口密度热力图，默认叠加在OpenStreetMap底图上，效果非常棒。

![](http://ohghnje4x.bkt.clouddn.com/image/170423/wmap_harvard.png)

让我特别感兴趣的是历史地图部分。上面赫然列着北宋、明、清的进士散点图和热力图。当把它们和当今人口密度热力图叠加显示，我们会惊讶地发现过去一千年来盛产进士的地方，几乎严丝合缝地对应着今天中国人口最稠密的地区。进士分布只能指示那个时代的财富分布。但把千年以来的进士之乡叠加起来（如同叠加起无数个财富变迁的历史断面），就能看到这些财富分布的残影。而古今之间的这种辉映，说明今天的人们依然在从宋以降的这种“地气”大格局中受惠。

右键这三个进士数据层，选择“Share Layer”，就能看到一个分享页面。除了视图本身，还有参考数据集的链接，可以导出多种格式。我们导一个Excel出来。Excel的名称是CBDB_exams_NSong_WGS84_kto.xls。

![](http://ohghnje4x.bkt.clouddn.com/image/170423/cbdb_data.png)

这就十分厉害了：
1. CBDB，是大名鼎鼎的哈佛中国历代人物传记数据库(China Biographical Database)。这个图层的基础，来自CBDB，可靠性就有保障了。这个数据库内容非常丰富，后面还会用到；
2. WGS84，说明坐标系用的是WGS-84坐标系，所以只要利用基于WGS-84的GIS工具，就不需要针对GCJ-02或BD-09做逆偏置了。

# 进士最大来源地？

先把北宋、明、清三个进士数据集都载入R。

```r
library(readr)
nsong.js <- read_csv("下载/CBDB_exams_NSong_WGS84_kto.csv")
ming.js <- read_csv("下载/CBDB_exams_Ming_WGS84_MbJ.csv")
qing.js <- read_csv("下载/CBDB_exams_Qing_WGS84_GWU.csv")
```

简单看一下哪个地方来的进士最多。

## 北宋

```r
sort(table(nsong.js$AddrChn), decreasing=TRUE)
```

| 序号 | 来源地  | 人数 |
|:----|:------|------:|
| 1 |  莆田 | 194 |
| 2 |  吳縣 | 121 |
| 3 |  閩縣 |  94 |
| 4 |  晉江 |  88 |
| 5 |  仙遊 |  69 |
| 6 |  侯官 |  62 |
| 7 |  開封 |  60 |
| 8 |  眉山 |  60 |
| 9 |  武進 |  51 |
|10 |  德興 |  50 |

大胡建第一名！这还不算，胡建里的第一名是莆田！

## 明朝

```r
sort(table(ming.js$AddrChn), decreasing=TRUE)
```

| 序号 | 来源地  | 人数 |
|:----|:-------|-------:|
| 1   | 莆田  | 109 |
| 2   | 鄞縣  | 40 |
| 3   | 華亭  | 37 |
| 4   | 吉水  | 33 |
| 5   | 慈溪  | 28 |
| 6   | 安福  | 25 |
| 7   | 平湖  | 22 |
| 8   | 吳縣  | 22 |
| 9   | 餘姚  | 21 |
| 10  | 常熟  | 20 |

到明朝，浙江赶上来了。但莆田还是第一名，并且是等于2-4名总和的那种第一名！

## 清朝

```r
sort(table(qing.js$AddrChn), decreasing=TRUE)
```

| 序号 | 来源地  | 人数 |
|:----|:-------|------:|
| 1    | 錢塘  | 93 | 
| 2    | 仁和  | 91 |
| 3    | 大興  | 60 |
| 4    | 吳縣  | 39 |
| 5    | 長洲  | 39 |
| 6    | 侯官  | 38 |
| 7    | 桐城  | 37 | 
| 8    | 歙縣  | 36 |
| 9    | 武進  | 36 |
| 10   | 歸安  | 35 |

到清朝，浙江兴盛依旧，而莆田却掉队看不见了。
事实上，莆田在科举史上的衰落是非常惊人的。我们可以把图画出来。

```r
library(dplyr)
library(ggplot2)
library(ggthemes)
jinshi <- do.call("bind_rows", list(nsong.js, ming.js, qing.js))
jinshi.putian <- jinshi[jinshi$AddrChn=='莆田',]
jinshi.putian <- dcast(jinshi.putian, EntryYear~., length)
jinshi.putian$EntryYear <- as.integer(jinshi.putian$EntryYear)
ggplot() + geom_bar(aes(EntryYear, .), stat='identity', data=jinshi.putian) + 
    theme_hc() + ggtitle("莆田进士数变迁") + xlab("年份") + ylab("人数")
```

![](http://ohghnje4x.bkt.clouddn.com/image/170423/PutianJinshi.png)

南宋、元代没有数据，所以是空白。明代自1560年代后，进士数就突然非常稀疏。原因很简单：这一年倭寇攻陷莆田并屠城。从此莆田文脉衰颓，再也没有复兴。

即使在500年前就已弃赛，把北宋、明、清三代的进士数合起来统计，莆田还是遥遥领先。

```r
sort(table(jinshi$AddrChn), decreasing=TRUE)
```

| 序号 | 来源地  | 人数 |
|:----|:-------|------:|
| 1  | 莆田  | 311 | 
| 2  | 吳縣  | 182 |
| 3  | 錢塘  | 157 |
| 4  | 閩縣  | 137 |
| 5  | 晉江  | 118 |
| 6  | 鄞縣  | 108 |
| 7  | 侯官  | 107 |
| 8  | 武進  | 103 |
| 9  | 仁和  | 99 |
| 10 | 歸安  | 72 |

所以，以后碰到“南蛮、南蛮”的地图炮可以硬怼——过于可笑了，以千年尺度来衡量，进士生源地的前十名都是南蛮。

# 地理可视化

接下来，我要把这些信息用ggplot2可视化到地图上，需要采集很多额外的GIS数据。

## 当代疆域

当代疆域作为底图，需要拿国/省边界数据。在R中，`mapdata`配合`maps`可以取到国界数据，但太老了（重庆都没有）。反正只是示意，精度要求不那么高，我们可以到开放数据平台Diva-GIS.org去找数据。

![](http://ohghnje4x.bkt.clouddn.com/image/170423/divagis.png)

需要分别下载CHN和TWN的地图数据（香港HKG和澳门MAC就不去下载了，反正都很小，不影响主要效果）。把CHN的Level1和TWN的Level0数据拼合起来，就大致妥了。

```r
cn.mapdata <- readOGR("下载/Regime_Bou/CHN_adm1.shp")
tw.mapdata <- readOGR("下载/Regime_Bou/TWN_adm0.shp")
p.chn <- ggplot() + geom_polygon(
    aes(long, lat, group=group), data=cn.mapdata, 
    fill='gray97', color='gray', linetype=2, size=0.2) + 
    geom_polygon(
        aes(long, lat, group=group), data=tw.mapdata, 
        fill='gray97', color='gray', linetype=2, size=0.2) +
    theme_minimal() + coord_map() 
```

这个p.chn对象后面可以反复复用。

![](http://ohghnje4x.bkt.clouddn.com/image/170423/ChnMap.png)


## 古代疆域

古代疆域就不太容易下载到现成的了，好在我们也有开源平台可以用：发现中国（webdog.cn）。这是一个基于WebGIS技术的公益网站，用户可以自己创建地图，最简单的玩法是衬一张历史地图底图，然后创建图层描点。地图数据以KML格式存储，可以导出来。KML本质是一种XML标记语言，Google地图就使用它。

![](http://ohghnje4x.bkt.clouddn.com/image/170423/webdog.png)

我们可以登进去，复制其他用户创建好的地图，然后导出来。导出的KMZ是KML的压缩包，Zip解压即可。

创建两个工作函数，把KML转化为list，然后从list中提取出坐标数据，并清理转化为数据框。

```r
get_xml_list <- function(xml){
    library(XML)
    xmlToList(xmlParse(xml))
}
get_coord_df <- function(lst, split="[ ,]", id=1){
    staging <- strsplit(lst, split)
    out <- data.frame(matrix(
        as.numeric(unlist(staging)), byrow=TRUE, ncol=2))
    names(out) <- c("long", "lat")
    out$id <- id
    return(out)
}
```

由于每张地图都有多个边界节点（大陆、海南、台湾），所以要串成list提出来。这就需要利用mapply了：提取、命名map_id、合并数据框，一气呵成。

```r
# 北宋
nsong.xml <- get_xml_list("~/下载/northSong.kml")
nsong.bou <- c(
    nsong.xml[[1]][[3]][[3]]$Polygon$outerBoundaryIs$LinearRing$coordinates,
    nsong.xml[[1]][[3]][[4]]$Polygon$outerBoundaryIs$LinearRing$coordinates)
nsong.bou <- mapply(get_coord_df, nsong.bou, id=1:2, SIMPLIFY=FALSE)
names(nsong.bou) <- 1:2
nsong.bou <- do.call('rbind', nsong.bou)
# 明朝
ming.xml <- get_xml_list("~/下载/ming.kml")
ming.bou <- c(
    ming.xml[[1]][[3]][[3]]$outerBoundaryIs$LinearRing$coordinates,
    ming.xml[[1]][[4]][[3]]$outerBoundaryIs$LinearRing$coordinates,
    ming.xml[[1]][[5]][[3]]$outerBoundaryIs$LinearRing$coordinates)
ming.bou <- mapply(get_coord_df, ming.bou, id=1:3, SIMPLIFY=FALSE)
names(ming.bou) <- 1:3
ming.bou <- do.call('rbind', ming.bou)
# 清朝
qing.xml <- get_xml_list("~/下载/qing.kml")
qing.bou <- c(
    qing.xml[[1]][[3]][[3]]$outerBoundaryIs$LinearRing$coordinates,
    qing.xml[[1]][[4]][[3]]$outerBoundaryIs$LinearRing$coordinates,
    qing.xml[[1]][[5]][[3]]$outerBoundaryIs$LinearRing$coordinates,
    qing.xml[[1]][[6]][[3]]$outerBoundaryIs$LinearRing$coordinates)
qing.bou <- mapply(get_coord_df, qing.bou, id=1:4, SIMPLIFY=FALSE)
names(qing.bou) <- 1:4
qing.bou <- do.call('rbind', qing.bou)
```

宋、明火德，咱涂个红色。清没有官宣自己什么德，但坊间一般认为丫是水德，咱给涂个黑色。

```r
p.song <- p.chn + geom_polygon(aes(long, lat, group=id), 
	data=nsong.bou, fill="red", alpha=0.1) +
    ggtitle("北宋疆域")
p.ming <- p.chn + geom_polygon(aes(long, lat, group=id), 
	data=ming.bou, fill="red", alpha=0.1) +
    ggtitle("明朝疆域")
p.qing <- p.chn + geom_polygon(aes(long, lat, group=id), 
	data=qing.bou, fill="black", alpha=0.1)+
    ggtitle("清朝疆域")
```

![](http://ohghnje4x.bkt.clouddn.com/image/170423/NSongMap.png)

![](http://ohghnje4x.bkt.clouddn.com/image/170423/MingMap.png)

![](http://ohghnje4x.bkt.clouddn.com/image/170423/QingMap.png)

完美。

## 进士来源地

```r
p.song + geom_point(aes(x_coord, y_coord), data=nsong.js, 
	color="blue", alpha=0.2, size=0.5) +
    ggtitle("北宋进士来源地")
p.song + stat_density_2d(aes(x_coord, y_coord, fill=..level..), 
    data=nsong.js, geom="polygon", alpha=0.5) +
    scale_fill_gradient(low="cyan", high="darkblue")+
    ggtitle("北宋进士来源地")
```

![](http://ohghnje4x.bkt.clouddn.com/image/170423/NSongJinshi1.png)

![](http://ohghnje4x.bkt.clouddn.com/image/170423/NSongJinshi2.png)

北宋的进士来源地呈现江浙、福建双巨头，江西、四川、河南三极的格局。元祐新旧党争，旧党分为朔、洛、蜀，正好对应于河南、四川这两极，而司马光特别瞧不起闽、楚之人，正是进士热图上特别炽热的福建、江西。之所以敌意这么重，就是因为这几个地方科举太厉害了，却聊不到一块儿去。

```r
p.ming + geom_point(aes(x_coord, y_coord), data=ming.js, 
	color="blue", alpha=0.2, size=0.5)+
    ggtitle("明朝进士来源地")
p.ming + stat_density_2d(aes(x_coord, y_coord, fill=..level..), 
	data=ming.js, geom="polygon", alpha=0.5) +
    scale_fill_gradient(low="cyan", high="darkblue")+
    ggtitle("明朝进士来源地")
```

![](http://ohghnje4x.bkt.clouddn.com/image/170423/MingJinshi1.png)

![](http://ohghnje4x.bkt.clouddn.com/image/170423/MingJinshi2.png)

到明代，双巨头格局不复存在，只剩下江浙一极独大，其余几个热点地区分别是北直隶、江西、福建、河南。四川从多极格局中完全消失，可见蒙元入侵影响之深远。天启中，东林党与阉党争权，前者恰好对应于江南单极，于是剩下几极（齐党、楚党、浙党）不得不寄身阉党羽翼下。

```r
p.qing + geom_point(aes(x_coord, y_coord), data=qing.js, 
	color="blue", alpha=0.2, size=0.5)+
    ggtitle("清朝进士来源地")
p.qing + stat_density_2d(aes(x_coord, y_coord, fill=..level..), 
	data=qing.js, geom="polygon", alpha=0.5) +
    scale_fill_gradient(low="cyan", high="darkblue")+
    ggtitle("清朝进士来源地")
```

![](http://ohghnje4x.bkt.clouddn.com/image/170423/QingJinshi1.png)

![](http://ohghnje4x.bkt.clouddn.com/image/170423/QingJinshi2.png)

清代继续沿袭江浙单极格局。福建进一步萎缩，而长江中下游的安徽、湖北、湖南、江西都呈现相当的强势，另一个强势的地区则是京畿的直隶、山东。

从进士热图来看：

1. 长江中下游的整体强势千年未减；
2. 福建、四川逐渐褪色;
3. 京畿总能额外占据一极，这一点，今天依然如故。

把历代数据叠起来看，更加清晰。江浙赣闽文脉相连，而另一个特别明显的现象是淮河流域有一整片真空——这里不怎么出进士，相应地比较适合出皇帝和武将。

```r
p.chn + geom_point(aes(x_coord, y_coord), data=jinshi, 
	color="blue", alpha=0.2, size=0.5)+
    ggtitle("北宋、明、清进士来源地")
p.chn + stat_density_2d(aes(x_coord, y_coord, fill=..level..), 
	data=jinshi, geom="polygon", alpha=0.5) +
    scale_fill_gradient(low="cyan", high="darkblue")+
    ggtitle("北宋、明、清进士来源地")
```

![](http://ohghnje4x.bkt.clouddn.com/image/170423/SongMingQingJinshi1.png)

![](http://ohghnje4x.bkt.clouddn.com/image/170423/SongMingQingJinshi2.png)

# 来源地的时间趋势

上面这些图都是把历时因素剔除后看的叠影效应。然而，这些来源地有没有什么时间上的变化趋势呢？

进士数据集包含EntryYear和AddrChn，但地名颗粒太细（县级），有必要归集到一级行政区。这就需要拿辅助数据了。

## CBDB数据库

CBDB数据库有两种离线版本：MS Access和SQLite。我用Linux，所以只能选后者。

![](http://ohghnje4x.bkt.clouddn.com/image/170423/harvard_cbdb.png)

下下来之后，用RSQLite来抽数据。我的用法不复杂，只要取出Addresses表就行。

```r
library(RSQLite)
con <- dbConnect(SQLite(), "下载/cbdb_sqlite.db")
addr <- dbReadTable(con, "ADDRESSES")
dbDisconnect(con)
```

数据表里有一列c_name_chn，对应于最小细度的地名，之后有belongs2_name，belongs3_name等上级政区名。merge一下，就能取到一级政区名了。

构造一个函数`get_stat`，来构成某朝代的政区字典，merge进进士数据集，并统计出时间分布。

```r
get_stat <- function(addr, jsdt, dyn.name, yr.begin, yr.end){
	addr.dyn <- addr[addr$belongs3_Name == dyn.name | 
					  addr$belongs4_Name == dyn.name |
                      addr$belongs5_Name == dyn.name,]
	addr.dyn <-addr.dyn[!is.na(addr.dyn$c_name_chn),]
	addr.dyn$Prov <- addr.dyn$belongs2_Name
	i <- which(addr.dyn$belongs4_Name == dyn.name)
	addr.dyn$Prov[i] <- addr.dyn$belongs3_Name[i]
	i <- which(addr.dyn$belongs5_Name == dyn.name)
	addr.dyn$Prov[i] <- addr.dyn$belongs4_Name[i]
	addr.dyn <- addr.dyn[!duplicated(addr.dyn$c_name_chn),]
	js <- merge(jsdt, addr.dyn[,c("c_name_chn", "Prov")], 
                by.x="AddrChn", by.y="c_name_chn", all.x=TRUE)
	js$Decade <- cut(as.numeric(js$EntryYear), seq(yr.begin, yr.end, 10))
	js$Decade <- factor(js$Decade, 
		labels=seq(yr.begin, yr.end, 10)[1:nlevels(nsong.js$Decade)])
	
	stat1 <- dcast(js, Prov~., length)
	stat1 <- stat1[order(stat1$., decreasing=TRUE),]

	stat <- dcast(js, Decade~Prov, length)
	stat <- melt(stat, id="Decade", stringsAsFactors=FALSE)
	stat$Decade <- as.integer(as.character(stat$Decade))
	stat$variable <- factor(stat$variable, levels=stat1$Prov)
	stat <- stat[!is.na(stat$variable) & stat$variable != 'NA',]
	return(stat)
}
```

这样做未必精确。有些地名匹配不到，有些一级行政区的名称有歧称。有时间还可以进一步审查，没时间就粗粗看个大概。

## 北宋

```r
palcols <- c(hc_pal()(10), tableau_color_pal()(10), wsj_pal()(6), 
		  canva_pal()(4), economist_pal()(10))
nsong.stat <- get_stat(addr, nsong.js, "宋朝", 960, 1130)
cols <- palcols[(1:nlevels(nsong.stat$variable))]
names(cols) <- levels(nsong.stat$variable)
ggplot(nsong.stat, aes(Decade, value, fill=variable))+
    geom_area(stat="identity", position="fill", alpha=0.75) + 
    scale_fill_manual(values=cols) + theme_minimal() +
    scale_y_continuous(labels=scales::percent)+
    xlab("年份") + ylab("比重") + ggtitle("北宋进士来源地比重变迁")+
    theme(legend.position="bottom")
```

![](http://ohghnje4x.bkt.clouddn.com/image/170423/NSongJinshiProp1.png)

![](http://ohghnje4x.bkt.clouddn.com/image/170423/NSongJinshiProp2.png)

宋初主要仍从后周控制区取士，之后京畿路和河北东路等北方地区比重急剧下跌，而福建路、两浙西、两浙东路等地进士比重不断攀升。这是北宋时地气南倾的明显表现。

## 明朝

明朝的情况比较特别，一级行政区会同时出现布政司、巡抚等并列。需要额外归并一下。

```r
ming.stat <- get_stat(addr, ming.js, "明朝", 1360, 1650)
ming.stat$variable <- gsub("(^.+)(布政司|巡撫|總督|留守司)$", "\\1", 
							ming.stat$variable)
ming.stat <- dcast(ming.stat, Decade ~ variable, sum)
ming.stat$Decade <- as.integer(ming.stat$Decade)
cols <- palcols[(1:nlevels(ming.stat$variable))]
names(cols) <- levels(ming.stat$variable)
ggplot(ming.stat, aes(Decade, value, fill=variable))+
    geom_area(stat="identity", position="fill", alpha=0.75) + 
    scale_fill_manual(values=cols) + theme_minimal() +
    scale_y_continuous(labels=scales::percent)+
    xlab("年份") + ylab("比重") + ggtitle("明朝进士来源地比重变迁")+
    theme(legend.position="bottom")
```

![](http://ohghnje4x.bkt.clouddn.com/image/170423/MingJinshiProp1.png)

![](http://ohghnje4x.bkt.clouddn.com/image/170423/MingJinshiProp2.png)

明代江西进士的比重一直在下降，而浙江、中都留守司（大体包括江苏和安徽）的比重明显升高。来自中都留守司辖区的进士比重在隆庆年间达到高峰，之后逐步下降。神宗后，来自山东、京师等北方地区的进士比重升高了。这也就是后来东林和阉党党争的人事基础。

## 清朝

```r
qing.stat <- get_stat(addr, qing.js, "清朝", 1640, 1910)
cols <- palcols[(1:nlevels(qing.stat$variable))]
names(cols) <- levels(qing.stat$variable)
ggplot(qing.stat, aes(Decade, value, fill=variable))+
    geom_area(stat="identity", position="fill", alpha=0.75) + 
    scale_fill_manual(values=cols) + theme_minimal() +
    scale_y_continuous(labels=scales::percent)+
    xlab("年份") + ylab("比重") + ggtitle("清朝进士来源地比重变迁")+
    theme(legend.position="bottom")
```

![](http://ohghnje4x.bkt.clouddn.com/image/170423/QingJinshiProp1.png)

![](http://ohghnje4x.bkt.clouddn.com/image/170423/QingJinshiProp2.png)

清代前期江苏和浙江占了大头，中后期逐渐下降，而一个醒目的变化是湖南、广东比重的上升。这些人构成清后期变法维新的主力。

# 姓氏

哪些姓氏的进士最多呢？

进士数据集有进士姓名。我们把这些姓名的首字提出来看一看（复姓就不管了。除非以后找一个语料库来细化）。

```r
library(stringr)
fam.name <- str_sub(jinshi$NameChn, 1,1)
sort(table(fam.name), decreasing=TRUE)
```

不知为什么，“張”写作了“瀳”，“陳”写作了“隇”。修正后，这个分布其实与整个大人群相差不大。

 王  李  瀳  隇  劉  吳  楊  黃  林  周  孫  方  朱  徐  酁  胡  許 <br />
630 477 375 373 306 242 207 196 184 182 166 164 143 138 127 124 108 <br />
 趙  沈  陸  何  汪  馮  葉  郭  江  錢  程  蔡  董  呂  范  曹  彭 <br />
 99  98  95  93  85  77  73  72  71  70  69  68  68  63  62  61  60 <br />
 宋  石  謝  曾  章  丁  高  梁  馬  唐  邵  余  蔣  姚  傅  潘  史 <br />
 60  59  59  58  58  57  56  55  55  55  53  49  48  48  47  47  46 <br />
 戴  蘇  袁  蕭  韓  毛  魏  金  俞  洪  羅  鄧  田  顧  秦  譚  夏 <br />
 43  42  42  40  39  39  37  36  36  35  35  34  32  31  31  31  31 <br />
 盧  秊  葛  黎  翁  熊  薛  孔  廖  歐  陶  詹  杜  侯  阮  嚴  鄒 <br />
 28  28  27  26  26  26  25  24  24  24  24  24  23  23  23  23  22 <br />
 倪  鮑  崔  柯  顬  上  虞  賈  闉  費  湯  萬  于  任  施  左  查 <br />
 21  20  20  20  20  20  20  19  18  17  17  17  17  16  16  16  15 <br />
 喬  姜  凌  柳  饒  易  應  莊  梅  齊  司  晏  尹  游  臧  晁  段 <br />
 15  14  14  14  14  14  14  14  13  13  13  13  13  13  13  12  12 <br />
 關  莫  龐  盛  祝  包  樊  管  華  武  伍  白  畢  聶  丘  韋  衛 <br />
 12  12  12  12  12  11  11  11  11  11  11  10  10  10  10  10  10

把前十名拿出来，做个地理分布可视化。

```r
jinshi$FamilyName <- fam.name
for (famName in c(
	"王", "李", "瀳", "隇", "劉", "吳", "楊", "黃", "林", "周")){
    pp <- p.chn + geom_point(aes(x_coord, y_coord), 
			data=jinshi[jinshi$FamilyName==famName,], 
            color="blue", alpha=0.2, size=0.5)+
        stat_density_2d(aes(x_coord, y_coord, fill=..level..), 
                        data=jinshi[jinshi$FamilyName==famName,], 
						geom="polygon", alpha=0.5) +
        scale_fill_gradient(low="cyan", high="darkblue")+
        ggtitle(paste0("北宋、明、清", fam.name, "姓进士来源地"))
    print(pp)
}
```

![](http://ohghnje4x.bkt.clouddn.com/image/170423/Jinshi_Wang.png)
![](http://ohghnje4x.bkt.clouddn.com/image/170423/Jinshi_Li.png)
![](http://ohghnje4x.bkt.clouddn.com/image/170423/Jinshi_Zhang.png)
![](http://ohghnje4x.bkt.clouddn.com/image/170423/Jinshi_Chen.png)
![](http://ohghnje4x.bkt.clouddn.com/image/170423/Jinshi_Liu.png)
![](http://ohghnje4x.bkt.clouddn.com/image/170423/Jinshi_Wu.png)
![](http://ohghnje4x.bkt.clouddn.com/image/170423/Jinshi_Yang.png)
![](http://ohghnje4x.bkt.clouddn.com/image/170423/Jinshi_Huang.png)
![](http://ohghnje4x.bkt.clouddn.com/image/170423/Jinshi_Lin.png)
![](http://ohghnje4x.bkt.clouddn.com/image/170423/Jinshi_Zhou.png)

哈，都是全国性的大姓，所以热点来自五湖四海。唯一的例外是“林”，基本上就是福建一地。

受此启发，看一下“钱”、“沈”、“陆”。果然是无锡钱、吴兴沈、苏州陆。

![](http://ohghnje4x.bkt.clouddn.com/image/170423/Jinshi_Qian.png)
![](http://ohghnje4x.bkt.clouddn.com/image/170423/Jinshi_Shen.png)
![](http://ohghnje4x.bkt.clouddn.com/image/170423/Jinshi_Lu.png)

姓氏地理分析意思相对不是那么大。差不多是各姓氏的随机抽样。

数据集和分析代码都存着。以后想到什么好玩的，还可以继续玩。

[完]

----

扫码关注我的的我的公众号

![](http://ohghnje4x.bkt.clouddn.com/QRcode.jpg)
