---
layout: post
title: 一次不太成功的搬砖（中）：rvest爬取法定传染病疫情月报数据
description: "用rvest爬取到卫计委法定传染病疫情月报的（坑爹）附件，通过构造工作函数，并行遍历，完成提取->清理->合并，获得一个真正可用的数据合集。"
author: madlogos
modified: 2017-4-15
tags: [数据]
comments: true
---

# 数据提取

现在，可以着手把存储在附件里的信息结构化提取出来了。但在这之前，还有一个硬骨头要啃。

**要把图片附件识别成文本。**

首先考虑OCR。但是Abbyy Finereader似乎没有Ubuntu版本。其他一些主流工具要钱。网上找到几个免费OCR工具，试用了下，转出来一堆乱码亲妈都唔识得。一怒之下，放了个大招：

**手工录入。**

![](http://ohghnje4x.bkt.clouddn.com/%E6%97%A0%E8%AF%9D%E5%8F%AF%E8%AF%B4.jpg)

这项工作很不好做，让我不禁怀疑起人生。但只有经过这样的磨练，才能对疾控系统的信息化水平有一个实操层面的认识。倘若遇到这方面的项目机会，**记得要把工程预算乘以3**。

图片方面的坑包括：
1. 有些图片附件分辨率低到了厚马赛克水准，别说OCR，钛金狗眼也认不出
2. 有些表格作为OLE对象内嵌到了Word文件里，当我满怀希望点进去才发现，这个内嵌对象竟仍是个图片
3. 有个别文件特别贴心地把表格割成两张图，插到了正文里

满脸辛酸地处理完了这些杂碎，把doc和xls存作docx和xlsx，接下来总算能把它们当成正常的xml来处理了。

> 有读者留言提到，这些数据其实都可以从公共卫生信息网申请到。没错。但是作为数据公开党，我对这种公共数据管制甚至收费牟利的做法非常不屑。这根本不符合如今的时代精神。本文提到的这些结构化数据文档，都已打包存到[七牛云](http://ohghnje4x.bkt.clouddn.com/image/170415/infect_dis_stat.zip)。人人都可以免费用。

## `docxtractr`

docx和xlsx本质上是一堆xml文件打包到zip里。所以2007以后的MS Office文件都更好处理，解包后按xml语法抽节点信息就是。不过人上有人，懒外有懒。我是不会用XML包做通用解的，敲那么多代码手指不会痛吗？

**除非人命关天，否则莫造新轮。**

我们可用`docxtractr`处理docx，`readxl`处理xlsx，`readr`处理csv。`docxtractr`有个特别贴心的函数`docx_extract_tbl`，直接把word正文里的表格提取成data.frame，就跟`html_table`一样。

## 提取工具函数

通过前面的苦力活，现在只剩下三种文件形态：csv、xlsx、docx。写一个通用方法来分类提取。

```r
library(docxtractr)
library(readxl)
library(readr)
readMsoTbl <- function(mso.file, header=TRUE) {
    file.type <- tolower(str_replace(
        mso.file, ".+\\.([^\\.]+)$", "\\1"))
    if (file.type == "csv"){
        invisible(read_csv(mso.file, col_names=header))
    }else if (file.type == "docx"){
        docx <- invisible(read_docx(mso.file))
        docx_extract_tbl(docx, 1, header=header)
    }else if (file.type == "xlsx"){
        invisible(read_excel(mso.file, col_names=header))
    }else{
        NULL
    }
}
```

然后用`lapply`跑个隐式循环，就把所有表格都以data.frame的形式提出来了，存为一个逼格李斯特(big list)。

```r
data <- lapply(list.files("~/infectdis", full.names=TRUE), 
               invisible(readMsoTbl))
```

# 数据清理

这样得到的数据虽然结构化了，但仍有很多问题。

1. 变量名都是X1, X2, ...， 需要转成数据原本的表头
2. 存在空行和空列
3. 数值列含有缺失值和数值文本混合值
4. 病名多样，比如“甲肝”和“甲型肝炎”本质上是一回事

可以分几步走：重新定义表头，然后舍弃/纠正不规范数值，最后归并同类病名。

## 值规范化

构造两个工作函数，然后lapply一轮就能把数值规范化：

- `redefCol`用来规范每张表格的表头。如当前用的是X1, X2, ...，就用首行替代。最后把变量名中的空格、星号都去掉
- `cleanTbl`用来去掉空列、空行，去掉“病名”列中的空格、星号、括号等，把发病数、死亡数两列的非数字字符都去掉

> 由于后面要用到并行计算，所以工作函数内要么显式引用加载包`stringr`等，要么在函数前声明其所在命名空间，如`stringr::str_detect()`。

```r
# 重定义首行
redefCol <- function(df){
    ## Arg
    ##     df: data.frame
    if (all(str_detect(colnames(df), "[Xx]\\d"))){
        colnames(df) <- df[1,]
        df <- df[2:nrow(df),]
    }
    colnames(df) <- str_replace_all(
        colnames(df), "\\s|\\*", "")
    return(df)
}

# 数据整形
cleanTbl <- function(df){
    ## Args
    ##    df: data.frame
    ##    dop: date of publication
    
    # 去掉空列、空行
    is.colallNA <- sapply(df, function(vec){
        all(is.na(vec)) | all(nchar(vec)==0)})
    is.rowallNA <- apply(df, 1, function(vec){
        all(is.na(vec)) | all(nchar(vec)==0)})
    o <- df[!is.rowallNA, !is.colallNA]
    # 去掉首列空格，名称规范化
    o[[1]] <- stringr::str_replace_all(
        o[[1]], "[\\s＊\\*（）]", "")
    # 确保发病和死亡都是整数
    invisible(lapply(2:3, function(i){
        o[[i]] <<- as.numeric(stringr::str_replace(
            o[[i]], "\\D", ""))
        o[[i]][is.na(o[[i]])] <<- 0
    }))
    return(as.data.frame(o))
}
```

`cleanTbl`函数内部用了好几个apply家族函数，可想而知肯定很慢。所以遍历data列表时，可以用一下并行计算`parallel`。

先要创建一个集群，利用`makeCluster`。这里声明构造4个集群，因为`detectCores()`会告诉系统，这台电脑有4核。少声明几个也无所谓。

```r
library(parallel)
cl <- makeCluster(getOption("cl.cores", detectCores()))
```

创建集群，就是为了用`parLapply`，这其实就是`lapply`的并行版本。原来是`snow`包里的。并行调用`cleanTbl`后，清干净的列表存为dat。尺寸上明显小了很多。

```r
dat <- parLapply(cl, dat, invisible(cleanTbl))
```

dat用日期命名，然后再用一次lapply隐式循环，遍历dat的同时为每张表新增一列DOP。这里要用超赋值符<<-。

```r
names(dat) <- str_replace(
    list.files("~/infectdis"), "^(.+)\\..+$", "\\1")
invisible(lapply(1:length(dat), function(i){
    dat[[i]]$DOP <<- as.Date(names(dat)[i])}))
```

清理完毕！最后调用`dplyr`的`bind_rows`，把这些列表中包裹的数据框提出来合并成一个大数据框。这个框就是后续分析的基础了。

```r
library(dplyr)
dat <- do.call("bind_rows", dat)
```

## 归并同类

首先，定义一个正则转化字典，然后遍历一遍，就把同类病名都归并了。

```r
dict <- data.frame(
    pattern=c(
        "^.*甲乙丙类.*$", "甲乙类传染病小计", 
        "丙类传染病合计", "([甲乙丙丁戊])肝", "^未分型$|未分型肝炎", 
        "其他", "人感染H7N9禽流感", "布病", "钩体病", "^.*出血热.*$", 
        "^.*斑疹伤寒.*$", "伤寒\\+副伤寒"), 
    replace=c(
        "合计", "甲乙类传染病合计", "丙类传染病小计", "\\1型肝炎",
        "肝炎未分型", "其它", "人感染高致病性禽流感", "布鲁氏菌病", 
        "钩端螺旋体病", "流行性出血热", "流行性和地方性斑疹伤寒",
        "伤寒和副伤寒")
)
# 按行遍历dict，将dat$病名中符合'pattern'的，替换为'replace'
apply(dict, 1, function(vec) {
    invisible(
        dat$病名 <<- str_replace(dat$病名, vec[1], vec[2]))
})
```

再然后，创建一个变量Class，标记甲、乙、丙三个分类。

```r
dat$Class <- NA
dat$Class[dat$病名 %in% c("霍乱", "鼠疫")] <- "甲类"
dat$Class[dat$病名 %in% c(
    "病毒性肝炎", "细菌性和阿米巴性痢疾", "伤寒和副伤寒", "艾滋病",
    "淋病", "梅毒", "脊髓灰质炎", "麻疹", "百日咳", "白喉", 
    "流行性脑脊髓膜炎", "猩红热", "流行性出血热", "狂犬病", 
    "钩端螺旋体病", "布鲁氏菌病", "炭疽", "流行性乙型脑炎",
    "疟疾", "登革热", "新生儿破伤风", "肺结核", "传染性非典型肺炎",
    "人感染高致病性禽流感", "血吸虫病", "甲型H1N1流感")] <- "乙类"
dat$Class[dat$病名 %in% c(
    "流行性感冒", "流行性腮腺炎", "风疹", "急性出血性结膜炎",
     "麻风病", "包虫病", "丝虫病", "其它感染性腹泻病", "手足口病",
    "流行性和地方性斑疹伤寒", "黑热病")] <- "丙类"
names(dat) <- c("病名", "发病数", "死亡数", "日期", "分类")
dat$分类 <- factor(dat$分类, levels=c("丙类", "乙类", "甲类"))
```

# 通用作图函数

接下来我计划做一系列面积图，简单看看疫情的时间分布有什么有趣之处。但每次整形一遍，再写一堆ggplot命令是很烦人的。我盘算了下，大约要跑十几张图，如果写个通用作图函数增加代码复用性，整体来说还是合算的。

**作为码农，不光要坚定地偷懒，还要偷得值。**

简单说来，这个函数可以接过一个初步分析结果数据框，根据指定的xvar、yvar、gvar来设置`geom_area()`的`aes`参数，再套用一下HighChart的主题。这样每次做图，只需要写一行代码就完事了。

下面的代码是本次分析可视化的最核心部分。

```r
library(ggplot2)
library(ggthemes)
makeTsPlot <- function(
    df, title, unit="4 months", xlab=xvar, ylab=yvar,
    xvar="日期", yvar="value", gvar="分类", 
    legend.position=c(0.6, 1.05)
){
    ## Arg:
    ##    df: data.frame, source data
    ##    title: plot title
    ##    unit: a num or date_breaks
    ##    xlab, ylab: x-axis y-axis caption
    ##    xvar, yvar, gvar: var name of x, y, group
    ##    legend.position: a value that ggplot2::theme() accepts
    
    if (inherits(df[,xvar], c("POSIXt", "Date"))){
        breaks <- seq(min(df[,xvar]), max(df[,xvar]), unit)
        labels <- format(breaks, "%m\n%y")
        min.mon <- sort(format(breaks,"%m"))[1]
        labels[!str_detect(labels, paste0("^", min.mon))] <- format(
            breaks[!str_detect(labels, paste0("^", min.mon))], "%m")
        labels <- str_replace(labels, "^0", "")
    }else if (is.numeric(df[,xvar])){
        breaks <- labels <- 
            seq(min(df[,xvar]), max(df[,xvar]), unit)
    }else{
        breaks <- labels <- unique(df[,xvar])
    }
    pal <- ggthemes_data$hc$palettes$default[c(1,3,2,4:10)]
    if (length(pal) < length(unique(df[,gvar]))){
        pal <- rep(pal, ceiling(
            length(unique(df[,gvar])) / length(pal)))
    }
    pal <- pal[seq_len(length(unique(df[,gvar])))]
    p <- ggplot(df, aes(eval(parse(text=xvar)), 
                   eval(parse(text=yvar)), 
                   color=eval(parse(text=gvar)), 
                   fill=eval(parse(text=gvar)))) +
        geom_area(alpha=0.25, position="stack") + 
        theme_hc() + 
        scale_fill_manual(
            guide=guide_legend(title=gvar), values=pal) +
        scale_color_manual(
            guide=guide_legend(title=gvar), values=pal) +
        theme(axis.ticks=element_line(linetype=0),
              legend.position=legend.position,
              legend.direction="horizontal") +
        xlab(xlab) + ylab(ylab) 
    if (inherits(df[,xvar], c("POSIXt", "Date"))) {
        p <- p + scale_x_date(breaks=breaks, labels=labels) +
            labs(title=title, subtitle=paste(
                format(min(df[,xvar]), "%Y-%m"),
                format(max(df[,xvar]), "%Y-%m"), sep="~"))
    }else if (is.numeric(df[,xvar])){
        p <- p + scale_x_continuous(breaks=breaks, labels=labels) +
            labs(title=title, subtitle=paste(
                min(df[,xvar]), max(df[,xvar]), sep="~"))
    }else{
        p <- p + scale_x_discrete(breaks=breaks, labels=labels) +
            labs(title=title, subtitle="")
    }
    p
}
```

利用这个函数，只要来个数据框，就能出图。此外也不失灵活性，部分美学效果可以自定义调整。

[待续]

----

扫码关注我的的我的公众号

![](http://ohghnje4x.bkt.clouddn.com/QRcode.jpg)
