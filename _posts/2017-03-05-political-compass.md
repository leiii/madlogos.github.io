---
layout: post
title: 朝左还是朝右？来看一份17万行的小样本数据
author: madlogos  
date: 2017/3/5
tags: [数据]
image:
  feature: http://ohghnje4x.bkt.clouddn.com/image/170305/1c06a139efd5c1bee92364b82dbfbef1.jpg
---

# 回味2016

![](http://ohghnje4x.bkt.clouddn.com/image/170305/2015113154244635.png)

2017年已经热火朝天地过掉两个月。现在回过头去评价2016年，显得反射弧特别长。但我仍禁不住要多回头看两眼，因为直到现在，我还不能完全理解它。

表面看，鹄年一声炮响，送来了英国脱欧和Trump，以及无数脍炙人口的自媒体段子。华尔街全球化秩序的崩裂，给国际政治投下两个巨大的阴影。这源自30年前苏_东_剧_变和中_国_改_开，全球化捡到两个巨大的劳动力要素价值洼地，于是轰鸣运转、财源滚滚。而今红利渐渐吃尽，而劳动生产率其实并没有什么革命性增长，资本回报率一天不如一天，旧模式不免穷途末路。而它带来的副产物（贫_富_分_化、难_民_危_机、恐_怖_主_义、债务危机）却开始反噬全球化主导经济体自身。所以在可以预计的未来，崩裂还将加速，阴影也将进一步扩大。

但崩裂成什么模样，并不好说。布雷顿森林体系崩裂，其实并没有闹出什么大浪来。但凡尔赛体系的崩裂，就直接导致了第二次世界大战。西晋帝国体系的崩溃，带来了长达五百年的大分裂。如果把回望尺度放大一点，2016年可能只是历史进程的左右摇摆周期中的一小部分。19世纪后期西方自由主义大发展，对内损害了平等，对外制造了殖民，结果殖民地红利吃尽后，在20世纪催生了两次世界大战，以及追求平等的共_运和反殖民运动。然而平等主义在吃完战后红利就逐渐陷入了极/权和经济乏力，于是新自由主义回潮。这波回潮绵延至今（构成了我们这代人迄今人生经历的主体），如今同样碰到了瓶颈，于是向平等转向的声音又高涨起来。每一次左右摇摆，几乎都伴随着战争、动乱和政经格局的洗牌。

每一次摇摆，大输家都是那些共识建设较差的社会。如果社会共识不能应付新的形势，或在形势变化面前发生断裂，都会增加内生动乱的风险。而赢家，则是把自身共识引领到世界的那个社会。

问题来了，

# 我们的社会共识到底是什么？

这个问题不太好回答。在美国社会，核心的议题可能是税收、种族、最低工资、医保、LGBT权利、堕_胎、持枪。而中国社会的公开核心议题是就业、房价、环境，不公开的核心问题是如何评价历史遗留问题。在很多问题上，一个复杂社会非但不能形成稳定共识，相反，会有严重的分歧和激烈的交锋。这是有益的：共识的凝聚无不是经过充分的探讨而来。

但中国目前的现实是，人们会自发地绕过很多有价值的议题，或者被限制公开讨论。立法过程中有民意征集环节，但多是不公开结果的。到底全社会怎么看待某个问题，很少得到公开的定量结果。

从整体上把握社会共识，需要集体对一系列核心议题投票，覆盖个人权利、政府权利、经济、文化等各个领域。2007年，北大未名BBS网友借鉴了著名的pol_itial compass，推出了一个政_治倾向调查。这个调查一度很火爆，到2014年累积了近百万条数据。但2015年，它被正式墙了。

在这短短七年，我能明晰地感受到整个社会的风气转向。资本对公共决策的渗透能力越来越强，社会淡是非、跪实利的取向很重。按照秦_晖的讲法，如今中国连自由主义者都不谈效率，转而改谈平等问题，可见这个社会右成了什么样子。但另一方面，社会治理的技术进步也很显著，带来诸多正面影响。这一切究竟是怎样潜移默化地发生的？真让人好奇。

未名调查的作者把2014年的17万条数据公布了出来，这提供了一个不可多得的有趣样本。

# 我们来看一看

打开R。先载入程序包和数据。


```r
sapply(c("RColorBrewer", "readr", "stringr", "data.table", "ggplot2", "ggthemes",
         "scatterplot3d"), require, character.only = TRUE, quietly = TRUE)
data <- read_csv("http://zuobiao.me/resources/2014data.csv")
format(object.size(data), units = "Mb")
```

data一共171830行、57列（包括50个问题、记录号、人口学信息），占92.2M内存。把它melt成hypercube，会变成一头食内存兽。所以用readr包读入，再转成data.table。


```r
dt <- melt(data[, -3], id = c("X1","X2", "性别", "出生年份", "年收入", "学历"))
dt <- merge(data.table(dt), data.table(meta), by = "variable", all.x = TRUE)
dt$Value <- str_length(dt$value) / 2
dt$Agree <- str_detect(dt$value, "反对")
dt$Value[dt$Agree] <- -1 * dt$Value[dt$Agree]
dt$Age <- 2014 - dt$`出生年份`
dt$年龄段 <- cut(dt$Age, c(0, 19.99, 24.99, 29.99, 34.99, 39.99, 44.99, 99))
format(object.size(dt), units = "Mb")
```

果然，达到了700多M之巨，直接堆在内存里能把这台恐龙机撑爆。

这套问卷一共50道题，1-20题偏政治，21-40偏经济，41-50偏文化。选项都是“强烈支持”(2)、“支持”(1)、“反对”(-1)、“强烈反对”(-2)，没有中立选项（与Political compass一样）。构建一个meta数据集，论述含义偏向集体主义/威权主义/民族主义记为-1(左)，偏向自由主义/普世主义记为1(右)。两列相乘，-2到2分别对应“左”、“偏左”、“偏右”、“右”。原问题太长，化简成4-5各自的“话题”。

更早的一个版本还有IP地址A、B段，可以用来判断来源地。现存的版本把这个字段去掉了（手动可惜）。


```r
meta <- data.frame(variable=names(data)[4:53])
meta$Class <- c(rep("政治", 20), rep("经济", 20), rep("文化", 10))
meta$Type <- c(
    -1, 1, 1, -1, -1, 1, 1, -1, -1, -1,  # 4 - 13
    -1, 1, 1, 1, -1, -1, 1, 1, -1, -1,   # 14 - 23
    -1, -1, -1, 1, -1, -1, 1, -1, 1, -1, # 24 - 33
    1, -1, -1, 1, -1, -1, 1, -1, 1, 1,   # 34 - 43
    1, -1, -1, -1, 1, -1, -1, -1, -1, 1  # 44 - 53
    )
qn <- c(
    "普选权", "人权与主权", "信息公开", "多党制", "言论自由", "自主招生", 
    "公开传教", "统一军训", "领土完整优先", "司法程序正义", "对外援助", 
    "丑化领袖", "人民自决", "媒体代言", "国家唯利", "武统台湾", "律师辩护", 
    "双重国籍", "西方有敌意", "运动举国体制", "最低工资", "改革成果分配", 
    "集体利益优先", "个人自由", "价格干预", "关税保护", "教育公立", "国企意义", 
    "控制房市价格", "补贴穷人", "富人优先服务", "富人公示财源", "劳资要素地位", 
    "国企私有化", "命脉国企", "资本原罪", "地权民有", "农业补贴", "外资待遇", 
    "市场垄断", "性自由", "为尊者讳", "重新尊儒", "艺术评判", "生育自由", 
    "周易八卦", "中医药", "汉字简化", "国学启蒙", "同性恋")
names(qn) <- 1:50
meta$Qn <- qn
knitr::kable(meta)
```

|variable                                                                           |Class | Type|Qn           |
|:----------------------------------------------------------------------------------|:-----|----:|:------------|
|如果人民没有受过民主教育，他们是不应该拥有普选权的。                               |政治  |   -1|普选权       |
|人权高于主权。                                                                     |政治  |    1|人权与主权   |
|发生重大社会安全事件时，即使认为信息公开会导致骚乱的风险，政府仍应该开放信息传播。 |政治  |    1|信息公开     |
|西方的多党制不适合中国国情。                                                       |政治  |   -1|多党制       |
|在中国照搬西方式的言论自由会导致社会失序。                                         |政治  |   -1|言论自由     |
|由高校自主考试招生比全国统一考试招生更好。                                         |政治  |    1|自主招生     |
|应该容许宗教人士在非宗教场所公开传教。                                             |政治  |    1|公开传教     |
|无论中小学生或大学生，都应参加由国家统一安排的军训。                               |政治  |   -1|统一军训     |
|国家的统一和领土完整是社会的最高利益。                                             |政治  |   -1|领土完整优先 |
|哪怕经历了违反程序规定的审讯和取证过程，确实有罪的罪犯也应被处刑。                 |政治  |   -1|司法程序正义 |
|国家有义务进行对外援助。                                                           |政治  |   -1|对外援助     |
|国家领导人及开国领袖的形象可以作为文艺作品的丑化对象。                             |政治  |    1|丑化领袖     |
|当法律未能充分制止罪恶行为时，人民群众有权自发对罪恶行为进行制裁。                 |政治  |    1|人民自决     |
|应当允许媒体代表某一特定阶层或利益集团发言。                                       |政治  |    1|媒体代言     |
|如果国家综合实力许可，那么中国有权为了维护自己的利益而采取任何行动。               |政治  |   -1|国家唯利     |
|条件允许的话应该武力统一台湾。                                                     |政治  |   -1|武统台湾     |
|律师即使明知被辩护人的犯罪事实也应当尽力为其进行辩护。                             |政治  |    1|律师辩护     |
|应该允许中国公民同时具有外国国籍。                                                 |政治  |    1|双重国籍     |
|以美国为首的西方国家不可能真正容许中国崛起成为一流强国。                           |政治  |   -1|西方有敌意   |
|国家应当采取措施培养和支持体育健儿在各种国际比赛场合为国争光。                     |政治  |   -1|运动举国体制 |
|最低工资应由国家规定。                                                             |经济  |   -1|最低工资     |
|中国改革开放以来的经济发展的成果都被一小群人占有了，大多数人没得到什么好处。       |经济  |   -1|改革成果分配 |
|在重大工程项目的决策中，个人利益应该为社会利益让路。                               |经济  |   -1|集体利益优先 |
|浪费粮食也是个人的自由。                                                           |经济  |    1|个人自由     |
|如果猪肉价格过高，政府应当干预。                                                   |经济  |   -1|价格干预     |
|应当对国外同类产品征收高额关税来保护国内民族工业。                                 |经济  |   -1|关税保护     |
|教育应当尽可能公立。                                                               |经济  |    1|教育公立     |
|国有企业的利益属于国家利益。                                                       |经济  |   -1|国企意义     |
|试图控制房地产价格的行为会破坏经济发展。                                           |经济  |    1|控制房市价格 |
|改善低收入者生活的首要手段是国家给予财政补贴和扶持。                               |经济  |   -1|补贴穷人     |
|有钱人理应获得更好的医疗服务。                                                     |经济  |    1|富人优先服务 |
|高收入者应该公开自己的经济来源。                                                   |经济  |   -1|富人公示财源 |
|靠运作资金赚钱的人对社会的贡献比不上靠劳动赚钱的人。                               |经济  |   -1|劳资要素地位 |
|与其让国有企业亏损破产，不如转卖给资本家。                                         |经济  |    1|国企私有化   |
|那些关系到国家安全、以及其他重要国计民生的领域，必须全部由国有企业掌控。           |经济  |   -1|命脉国企     |
|资本积累的过程总是伴随着对普通劳动人民利益的伤害。                                 |经济  |   -1|资本原罪     |
|私人应当可以拥有和买卖土地。                                                       |经济  |    1|地权民有     |
|政府应当采用较高的粮食收购价格以增加农民收入。                                     |经济  |   -1|农业补贴     |
|在华外国资本应享受和民族资本同样的待遇。                                           |经济  |    1|外资待遇     |
|市场竞争中自然形成的垄断地位是无害的。                                             |经济  |    1|市场垄断     |
|两个成年人之间自愿的性行为是其自由，无论其婚姻关系为何。                           |文化  |    1|性自由       |
|不应公开谈论自己长辈的缺点。                                                       |文化  |   -1|为尊者讳     |
|现代中国社会需要儒家思想。                                                         |文化  |   -1|重新尊儒     |
|判断艺术作品的价值的根本标准是看是不是受到人民大众喜爱。                           |文化  |   -1|艺术评判     |
|即使有人口压力，国家和社会也无权干涉个人要不要孩子，要几个孩子。                   |文化  |    1|生育自由     |
|周易八卦能够有效的解释很多事情。                                                   |文化  |   -1|周易八卦     |
|中国传统医学对人体健康的观念比现代主流医学更高明。                                 |文化  |   -1|中医药       |
|汉字无需人为推行简化。                                                             |文化  |   -1|汉字简化     |
|应当将中国传统文化的经典作品作为儿童基础教育读物。                                 |文化  |   -1|国学启蒙     |
|如果是出于自愿，我会认可我的孩子和同性结成伴侣关系。                               |文化  |    1|同性恋       |

## 人口学代表性？

讲真，不怎么样。大多数都是30岁以下的小年轻，大学生和研究生比例特别高，低收入比例近一半。这只能说明一点：参与的主力军是在校大学生。而这个群体只占中国居民的10%都不到。


```r
calc <- dcast(data.table(dt), X1 ~ as.numeric(variable), mean, 
              value.var = "Value", na.rm = TRUE)
calc$`文化` <- rowMeans(calc[, 42:51])
calc$`经济` <- rowMeans(calc[, 22:41])
calc$`政治` <- rowMeans(calc[, 2:21])
data <- merge(data.table(data), data.table(calc[, c(1, 52:54)]), by = "X1")
```


```r
data$年龄 <- 2014 - data$出生年份
data$年龄组 <- cut(data$年龄, c(0, 19.99, 24.99, 29.99, 34.99, 39.99, 44.99, 99))
age.gender <- dcast(data, 性别 + 年龄组 ~., length, value.var = "X1")
age.gender <- age.gender[complete.cases(age.gender),]
age.gender$比重 <- age.gender$`.`/ sum(age.gender$`.`)
age.gender$比重[age.gender$性别 == "M"] <- - age.gender$比重[age.gender$性别 == "M"]
age.gender$性别 <- factor(age.gender$性别, levels=c("M", "F"))
theme_new <- function(){
    theme(axis.ticks.x = element_line(linetype = 0),
          axis.ticks.y = element_line(linetype = 0),
          panel.grid.major.y = element_line(size = 0.2))
}

ggplot(age.gender, aes(年龄组, 比重, group=性别)) + 
    geom_bar(aes(fill=性别), stat="identity", position="stack") +
    scale_y_continuous(breaks = c(-0.4, -0.3, -0.2, -0.1, 0, 0.1, 0.2),
                       labels=paste0(c(40, 30, 20, 10, 0, 10, 20), "%")) +
    theme_hc() + theme_new() + coord_flip()
```

![](http://ohghnje4x.bkt.clouddn.com/image/170305/Rplot.png)


```r
educ <- dcast(data, 学历  ~., length, value.var = "X1")
educ <- educ[complete.cases(educ),]
educ$比重 <- educ$. / sum(educ$.)
educ$学历 <- factor(educ$学历, levels=c(
    "初中及以下", "高中", "大学", "研究生及以上"))
educ <- educ[order(educ$学历),]
ggplot(educ, aes("", 比重, fill=学历)) + 
    geom_bar(stat="identity", position="stack", width=1, color="white") +
    geom_label(aes(x=1.75, y=1-cumsum(比重)+比重/2, label=scales::percent(比重))) +
    coord_polar(theta="y") + scale_y_continuous(labels=scales::percent) +
    theme_hc() + theme_new()
```

![](http://ohghnje4x.bkt.clouddn.com/image/170305/Rplot01.png)


```r
incm <- dcast(data, 年收入 ~., length, value.var = "X1")
incm <- incm[complete.cases(incm),]
incm$年收入 <- factor(incm$年收入, levels=c(
    "0-25k", "25k-50k", "50k-75k", "75k-100k", "100k-150k", "150k-300k",
    "300k+"))
incm$比重 <- incm$. / sum(incm$.)
incm <- incm[order(incm$年收入),]
ggplot(incm, aes("", 比重, fill=年收入)) + 
    geom_bar(stat="identity", position="stack", width=1, color="white") +
    geom_label(aes(x=1.75, y=1-cumsum(比重)+比重/2, label=scales::percent(比重))) +
    coord_polar(theta="y") + scale_y_continuous(labels=scales::percent) +
    theme_hc() + theme_new()
```

![](http://ohghnje4x.bkt.clouddn.com/image/170305/Rplot02.png)

可以猜测，这个群体会比普通中国人文化上更偏右一点，但政治和经济立场却未必。

## 三个维度上的得分分布？

还是比较正态的。政治几乎不偏不倚，经济和文化都轻微偏左。网上其他一些对此数据集的分析认为结果偏左，实际不然。


```r
ggplot() + theme_hc() + ggtitle("政治倾向均分") +
    geom_histogram(aes(政治), data = data, bins = 19, fill = hc_pal()(5)[1],
                   color = "white") +
    geom_vline(xintercept = mean(data$`政治`, na.rm = TRUE)) +
    theme_new()
```

![](http://ohghnje4x.bkt.clouddn.com/image/170305/Rplot04.png)


```r
ggplot() + theme_hc() + ggtitle("经济倾向均分") +
    geom_histogram(aes(经济), data = data, bins = 19, fill = hc_pal()(5)[3],
                   color = "white") +
    geom_vline(xintercept = mean(data$`经济`, na.rm = TRUE)) +
    theme_new()
```

![](http://ohghnje4x.bkt.clouddn.com/image/170305/Rplot05.png)


```r
ggplot() + theme_hc() + ggtitle("文化倾向均分") +
    geom_histogram(aes(文化), data = data, bins = 19, fill = hc_pal()(5)[4],
                   color = "white") +
    geom_vline(xintercept = mean(data$`文化`, na.rm = TRUE)) +
    theme_new()
```

![](http://ohghnje4x.bkt.clouddn.com/image/170305/Rplot06.png)


分解每个问题，会有更细的发现。

- 在经济控制i问题上，平均偏左，在市场问题上，平均偏右。

- 在尊儒/生育自由/汉字简化问题上，显著偏左。其余文化问题，普遍右倾。

- 在政务治理问题上，普遍左倾；在个人_权利问题上，普遍右倾。唯一较偏右的，是针对“运动举国体制”的态度。

但同样是国企话题，很多人反对国企私有化，但同时又赞成国企垄断国计民生命脉产业。这种矛盾很有意思。一些口号化的问题，容易激起下意识的回答。毕业后经历逐渐丰富，很多原本遥远的问题变得与切身利益攸关，感受更为真切，往往立场就会转变。


```r
summ <- merge(dcast(dt, as.numeric(variable) + Qn ~ ., c(mean, sd),
                    value.var = "Value", na.rm=TRUE), 
              meta, by = "Qn")
setorder(summ, variable.x)
summ <- summ[,c(1, 2, 3, 4, 6)]
names(summ) <- c("话题", "i", "均值", "标准差", "分类")
setorder(summ, 分类, -均值)
summ$话题 <- factor(summ$话题, levels=summ$话题)
ggplot() + geom_point(aes(话题, 均值, color = 分类), data = summ) + 
    geom_errorbar(aes(ymin = 均值-标准差, ymax = 均值+标准差, x = 话题), 
                  data = summ, color = "darkgray") +
    ggtitle("各问题均分及标准差") + geom_hline(yintercept=0, color = "darkgray") +
    coord_flip() + theme_hc() + theme_new()
```

![](http://ohghnje4x.bkt.clouddn.com/image/170305/Rplot08.png)

## 三个维度的关联？

关联很紧密，政治、文化、经济是正相关的。从下面这个图可以发现，男性的倾向性打分平均而言比女性更极端（偏离零点）。这17万个散点在三维空间里形成一个完美正态的橄榄球。


```r
p3d.lm <- with(data, lm(政治 ~ 文化 + 经济))
p3d <- with(data, scatterplot3d(
    文化, 经济, 政治, pch='', highlight.3d = FALSE, angle=120, type='h',
    main = paste("政治 =", round(p3d.lm$coefficients[2], 2), "* 文化 +", 
                 round(p3d.lm$coefficients[3], 2), "* 经济 +",
                 round(p3d.lm$coefficients[1], 2)), color = "gray95",
    col.axis="gray"))
p3d$points3d(data$文化[data$性别 == "M"], data$经济[data$性别 == "M"],
             data$政治[data$性别 == "M"], col = rgb(0, 0.75, 1, 0.025), pch = 20)
p3d$points3d(data$文化[data$性别 == "F"], data$经济[data$性别 == "F"],
             data$政治[data$性别 == "F"], col = rgb(1, 0.388, 0.278, 0.01), pch = 20)
p3d$plane3d(p3d.lm)
legend(p3d$xyz.convert(2,0,3), col=c("deepskyblue", "tomato"), pch=19,
       legend = c("M", "F"), border=NULL)
```

![](http://ohghnje4x.bkt.clouddn.com/image/170305/Rplot03.png)

对问题得分相关矩阵作层次聚类，大体可以分成四簇。其中，“富人公示财富来源”独立构成一簇。看来大家对此问题的态度比较矛盾。


```r
cor = (cor(calc[,2:51]))
colnames(cor) = qn
rownames(cor) = qn
plot(hclust(dist(cor)), sub="", xlab="", cex=0.6)
```

![](http://ohghnje4x.bkt.clouddn.com/image/170305/Rplot14.png)

## 不同人群的意见

进一步看。把学历、性别、年收入、年龄段四个人口学变量分别拿出来比较。

- 政治上，不同人群的差异不大，都是中偏右。

- 经济上，不同人群都偏左。收入越高的，越左倾（神奇）。年龄越大的越左（45岁以上的相反）。学历越低的越左（研究生以上相反）。

- 文化上整体仍偏左。学历越高越右，年收入越高越左（神奇），年龄越大越右（神奇，看来选择偏倚不小）。女性明显更左（偏集体/威权/平等）。


```r
summ <- lapply(c("性别", "年龄段", "年收入", "学历"), function(var){
    d = dcast(dt, as.formula(paste("Class +", var, "~.")), 
                             mean, value.var='Value', na.rm=TRUE)
    d$Attr = var
    names(d) = c("Class", "Level", "Mean", "Attr")
    return(d)
})
summ <- do.call('rbind', summ)
summ <- summ[!is.na(summ$Level) & summ$Level != "NULL",]
names(summ) <- c("分类", "水平", "均值", "标签")
summ$水平 <- factor(summ$水平, levels=c(
    "M", "F", "(0,20]", "(20,25]", "(25,30]", "(30,35]", "(35,40]",
    "(40,45]", "(45,99]", "0-25k", "25k-50k", "50k-75k", "75k-100k", 
    "100k-150k", "150k-300k", "300k+", "初中及以下", "高中", "大学", 
    "研究生及以上"))
cols <- c(scales::hue_pal()(2), brewer.pal(7, "Oranges"), 
          brewer.pal(7, "Greens"), brewer.pal(4, "Blues"))
names(cols) <- c(
    "M", "F", "(0,20]", "(20,25]", "(25,30]", "(30,35]", "(35,40]",
    "(40,45]", "(45,99]", "0-25k", "25k-50k", "50k-75k", "75k-100k", 
    "100k-150k", "150k-300k", "300k+", "初中及以下", "高中", "大学", 
    "研究生及以上")
ggplot() + geom_point(aes(标签, 均值, color = 水平), data = summ) + 
    ggtitle("分类均分比较") + geom_hline(yintercept=0, color = "darkgray") +
    facet_grid(.~分类) + scale_color_manual(values=cols) +
    coord_flip() + theme_hc() + theme_new()
```

![](http://ohghnje4x.bkt.clouddn.com/image/170305/Rplot07.png)

细分到具体问题，就更神奇了。

- 收入越高，越认同“西方对中国有敌意”、“国家应当进行价格干预”、“周易八卦有用”、“应当为尊者讳”、“武_统_台湾”、“统一军训”、“领土完整是社会优先利益”，越反对“汉字简化”。

- 收入越低，越反对“农业补贴”、“国家制定最低工资”、“国企应掌握命脉”、“可以丑_化_领_袖”。


```r
summ <- merge(dcast(dt, as.numeric(variable) + Qn + 年收入 ~ ., c(mean, sd),
                    value.var = "Value", na.rm=TRUE), 
              meta, by = "Qn")
setorder(summ, variable.x)
summ <- summ[summ$年收入 %in% c(
    "0-25k", "25k-50k", "50k-75k", "75k-100k", "100k-150k", "150k-300k",
    "300k+"), c(1:5, 7)]
names(summ) <- c("话题", "i", "年收入", "均值", "标准差", "分类")
setorder(summ, 分类, -均值)
summ$话题 <- factor(summ$话题, levels=unique(summ$话题))
summ$年收入 <- factor(summ$年收入, levels=c(
    "0-25k", "25k-50k", "50k-75k", "75k-100k", "100k-150k", "150k-300k",
    "300k+"))
ggplot() + geom_point(aes(话题, 均值, color = 年收入), data = summ) + 
    ggtitle("不同收入组各问题均分") + geom_hline(yintercept=0, color = "darkgray") +
    geom_vline(xintercept=c(20.5, 30.5), color="darkgray") + 
    scale_color_brewer(type="seq", palette = "Blues") +
    coord_flip() + theme_hc() + theme_new()
```

![](http://ohghnje4x.bkt.clouddn.com/image/170305/Rplot11.png)

- 学历越高，越认同：“应当控制房价”、“西方对中国有敌意”、“领土完整是社会优先利益”、“武统台湾”、“自主招生好于统招”、“富人应公示财产来源”，越反对“对外援助”、“汉字简化”。

- 学历越低，越反对：“农业补贴”、“劳动比资本高贵”、“丑化领袖”、“允许双重国籍”、“地权人民所有”。


```r
summ <- merge(dcast(dt, as.numeric(variable) + Qn + 学历 ~ ., c(mean, sd),
                    value.var = "Value", na.rm=TRUE), 
              meta, by = "Qn")
setorder(summ, variable.x)
summ <- summ[summ$学历 %in% c(
    "初中及以下", "高中", "大学", "研究生及以上"), c(1:5, 7)]
names(summ) <- c("话题", "i", "学历", "均值", "标准差", "分类")
setorder(summ, 分类, -均值)
summ$话题 <- factor(summ$话题, levels=unique(summ$话题))
summ$学历 <- factor(summ$学历, levels=c(
    "初中及以下", "高中", "大学", "研究生及以上"))
ggplot() + geom_point(aes(话题, 均值, color = 学历), data = summ) + 
    ggtitle("不同学历各问题均分") + geom_hline(yintercept=0, color = "darkgray") +
    geom_vline(xintercept=c(20.5, 30.5), color="darkgray") + 
    scale_color_brewer(type="seq", palette = 'Greens') +
    coord_flip() + theme_hc() + theme_new()
```

![](http://ohghnje4x.bkt.clouddn.com/image/170305/Rplot12.png)

- 年龄越大，越反对：“国企破产不如私有化”、“自然形成的市场垄断”、“尊儒”、“补贴穷人”、“资本有原罪”、“婚外性自由”、“多党制”、“可以丑化领袖”、“用国学启蒙”，越支持：“子女同性恋”。

- 年龄越小，越反对：“国企应控制经济命脉”、“双重国籍”、“浪费粮食也是个人自由”，越支持：“补贴穷人”、“尊儒”、“为尊者讳”、“用国学启蒙”。


```r
summ <- merge(dcast(dt, as.numeric(variable) + Qn + 年龄段 ~ ., c(mean, sd),
                    value.var = "Value", na.rm=TRUE), 
              meta, by = "Qn")
setorder(summ, variable.x)
summ <- summ[summ$年龄段 %in% c(
    "(0,20]", "(20,25]", "(25,30]", "(30,35]", "(35,40]", "(40,45]", "(45,99]"),
    c(1:5, 7)]
names(summ) <- c("话题", "i", "年龄段", "均值", "标准差", "分类")
setorder(summ, 分类, -均值)
summ$话题 <- factor(summ$话题, levels=unique(summ$话题))
summ$年龄段 <- factor(summ$年龄段, levels=c(
    "(0,20]", "(20,25]", "(25,30]", "(30,35]", "(35,40]", "(40,45]", "(45,99]"))
ggplot() + geom_point(aes(话题, 均值, color = 年龄段), data = summ) + 
    ggtitle("不同年龄段各问题均分") + geom_hline(yintercept=0, color = "darkgray") +
    geom_vline(xintercept=c(20.5, 30.5), color="darkgray") + 
    scale_color_brewer(type="seq", palette = "Oranges") +
    coord_flip() + theme_hc() + theme_new()
```

![](http://ohghnje4x.bkt.clouddn.com/image/170305/Rplot10.png)

- 多数政治问题上，女性更左（更保守）。

- 多数文化问题上，男性更右（自由主义）。

- 多数经济问题上，女性偏左。在补贴穷人问题上，男性明显偏右。


```r
summ <- merge(dcast(dt, as.numeric(variable) + Qn + 性别 ~ ., c(mean, sd),
                    value.var = "Value", na.rm=TRUE), 
              meta, by = "Qn")
setorder(summ, variable.x)
summ <- summ[summ$性别 %in% c("M", "F"), c(1:5, 7)]
names(summ) <- c("话题", "i", "性别", "均值", "标准差", "分类")
setorder(summ, 分类, -均值)
summ$话题 <- factor(summ$话题, levels=unique(summ$话题))
summ$性别 <- factor(summ$性别, levels=c("M", "F"))
ggplot() + geom_point(aes(话题, 均值, color = 性别), data = summ) + 
    geom_line(aes(话题, 均值, group = 性别, color = 性别), data = summ, alpha = 0.25) +
    ggtitle("男女各问题均分") + geom_hline(yintercept=0, color = "darkgray") +
    geom_vline(xintercept=c(20.5, 30.5), color="darkgray") +
    coord_flip() + theme_hc() + theme_new()
```

![](http://ohghnje4x.bkt.clouddn.com/image/170305/Rplot09.png)

## ... So?

我的政治、经济、文化倾向分别是0、0、0.4，十分中庸。2007年至今，我从一个文化自由主义右派慢慢向左靠。在Pol_itical compass上，类似的题目，我的经济和文化倾向分别是-5.13和0.67（满分10）。中国的右倾放到美国就成了一个自由主义左派。而中国的保守主义左派和美国的保守主义右派很多方面是互为镜像的。

单从上面这3年前的17万条数据来看，并没有像预想的那样偏，多数人是中间派，符合我们的日常经验，某种意义上也符合官方的期待。

在这个平均受教育水平较高的样本里，

- 政治倾向分歧不大；

- 经济上的分歧，体现在高收入者和年长者更追求“平等”；

- 文化观念上的分歧较大，年轻人更左。

如何理解中国语境下的“左”和“右”？

某种意义上，不同于欧美的左右之分，中国的保守/民族主义/左派倾向于维护现状，自由/普世主义/右派倾向于社会改革。所以现在看到的这些结果，意味着年轻人、女性和高学历者更不倾向于体制变革、更离弃普_世主义，也更倾向民族主义和自利主义。所以不难理解为何Trump的上台，会在这些从未受白左多元主义和政治正确路线伤害的中国的年轻人中引起一片叫好。

拿这个样本外推到全体大学生，乃至全体中国人，有相当大的困难。何况，从态度到公共决策还隔着十万八千里。所以单凭这些数据，很难推出更进一步的坚实结论。但是已有的发现足够引起适度的不安了。

回到最初的问题：

## 如果时局剧变，我们准备好了吗？

军事和经济上或许准备好了七八分。至于社会精神层面准备得如何，就不知道了。

但很大可能，我猜，中国社会会以迅速朝旧传统和民族主义回滚的方式来应对。新世代越发把新自由主义的普世精神当成笑话，目前来看，也并没有兴趣发展出一套独立的普世价值来替代。官方倒是很有这个兴趣，但目前的火候，成不了。

一个碎片化的纷争世界隐然可期了。

[完]

扫码关注我的的我的公众号

![](http://ohghnje4x.bkt.clouddn.com/QRcode.jpg)
