---
layout: post
title: 一次不太成功的搬砖（上）：爬取法定传染病疫情月报数据
description: "卫计委每月发布全国法定传染病疫情公告，包含甲乙丙三类传染病的发病和死亡统计。本打算趁清明假期（什么鬼），用rvest包写个简单的脚本，爬下来分析玩。不料遇到许多莫名其妙的坑。最后，好好一个【爬虫工程】变成了【批处理下载】，十分胸闷。拜托卫计委要继续提高自己的姿势水平。"
author: madlogos
modified: 2017-4-7
tags: [数据]
comments: true
---

# 带怀旧色彩的源起

清明节跑去一个休闲浴场~~鬼混~~，在电影厅懒散地看掉了《生化危机6》。场地很豪华（但我就是不透露门牌地址），然而剧情不怎么样——女主光环实在太亮了。倒是病毒-丧尸-疫苗的急性传染病建模设定引起了我的一些职业回忆。

毕业后，我曾在基层疾控中心干过一年多，主要做疫苗接种规划和传染病控制。除了定期不定期地出外勤下现场，就是统计数字、写报告、汇编材料。这些数字沿着行政金字塔的梯级层层上卷，最终汇入国家卫生部疾控局官方报表的大海中。

说是大海，视觉上其实就是类似这样的一张表格：

![法定传染病统计表](http://ohghnje4x.bkt.clouddn.com/image/170404/example_infectdis_report.png)

一晃很多年过去了。籍着这个由头，我又登上了卫生部（现在叫卫计委了，早晚改回卫生部）的官网，那感觉就像——拜会一个久寓故居，新近敲了墙、刷了房门的老派的朋友。那些月报还原封不动，化石一样静静地躺在信息动态里。

![传染病控制动态](http://ohghnje4x.bkt.clouddn.com/image/170404/nhfpc_infectdis_news.png)

![法定传染病月报](http://ohghnje4x.bkt.clouddn.com/image/170404/example_infectdis_reporttxt.png)

这种格式报告，行文和结构都很固定，特别适合用机器人来自动生成。比如最新这期，正文就包括了发病、死亡合计总数，以及甲乙丙类各自的发病、死亡数。明细数据放在附表里。掐指一数，从2004年到现在，卫计委也积攒了140多份月报，不少了。何不爬下来看看？

所以，尽管当时身还在浴场，但心在砖场了，已经！

# 搬砖设想

搬砖虽然是个贱活儿，但也要讲技巧。好在从技术角度，爬这些页面是再简单不过的事。只需要两步就完了：

1. 把所有月报页的链接抓到
2. 顺着这些链接把所有页面源码都爬下来

更好的消息是所有页面都是静态的。所以只要用`rvest`就够了，整页爬下来，所有信息就包含在html里面（事实上并不是）。

爬到所有月报页后，解析内容又是两步走：

1. 把正文里的发病/死亡总数抽出来，跑个时序图看看周期性
2. 把附表里的内容抽出来，分病种跑些分析

Easy as a pie！等我一盏茶的功夫，我去去就来。（结果茶馊了都没能回来）

# 爬目录

“传染病预防控制”这个分类的URL是很有规律的。第一页是http://www.nhfpc.gov.cn/jkj/s2907/new_list.shtml，第二页就是http://www.nhfpc.gov.cn/jkj/s2907/new_list_2.shtml。也就是说23个目录页拼一下就出来了：

```r
urls <- paste0(
    "http://www.nhfpc.gov.cn/jkj/s2907/new_list", 
    c("", paste0("_", 2:23)), ".shtml")
```

## `rvest`

有了URL，就可以爬源码了。当然可以把网页当文本，直接`readLines`，然后拿`XML`包写解析规则。但我们学R图什么？还不就是**免费+有很多包方便偷懒**？对于静态网页，当然毫不犹豫`rvest`。

`rvest`的核心函数是`read_html`、`html_nodes`、`html_text`和`html_table`。

- `read_html`很好理解，把页面读进来。这个页面会被封装为一个`xml_nodes`对象。
- `html_nodes`则负责从`xml_nodes`对象中提取某个节点的内容，封装成`xml_nodeset`对象。
- 进一步，如果要把里面的内容都当做文本提出来，用`html_text`。
- 有表格（<table>...</table>）的话，用`html_table`，直接输出梦寐以求的data.frame。

唯一费解的概念也就是“节点”。但只要对html和xml略有了解，就很容易理解。一个html文件的典型结构是

```html
<html>
  <head>
    <title>...</title>
  </head>
  <body>
    <h1>...</h1>
  </body>
</html>
```

从缩进结构可以看出，`html`是根节点，下一级是两个元素子节点`head`和`body`，`head`的子节点是`title`，`body`的子节点是`h1`，它们还可以有文本、属性或注释子节点。以此类推。

怎么看页面节点结构呢？用Chrome访问目录页，F12查看文档结构，或者右键“查看页面源代码”。

每一篇信息动态都在一个列表节点<li>里，最终都被包进一个无序列表父节点<ul>里。这个无序列表元素的类型是“zxxx_list”（果然还是万能的拼音首字母命名法，“资讯信息”）。所以拿到这个节点后，提取目录信息就很简单了：<a>节点内有标题和链接，<span ml>节点内有发布日期。

![网页源码](http://ohghnje4x.bkt.clouddn.com/image/170404/sourcecode_dir.png)

```r
library(rvest)
# 构造一个提取单个目录页信息的函数
getTOC <- function(url){
  ## Args
  ##   url: 单个网页/网址

    # 读取网页，构成xml_nodes
    html <- read_html(url)
    # 从html对象提取ul.zxxx_list节点，只要第一个元素
    cast <- html_nodes(html, "ul.zxxx_list")[[1]]
    # 从cast中提取所有<a>节点信息
    lists <- html_nodes(cast, "a")
    # 从cast中提取所有<ml>节点信息
    dop <- html_nodes(cast, "span.ml")
    # 最后得到一个3列数据框，包括链接、标题和发布日期
    return(data.frame(
        href=html_attr(lists, "href"),
        title=str_trim(html_text(lists)),
        dop=html_text(dop),
        stringsAsFactors=FALSE))
}
# 遍历urls，循环运行getTOC
toc <- lapply(urls, getTOC)
library(dplyr)
# 将23个数据框列表合并起来
toc <- do.call("bind_rows", toc)
```

当然，这些动态不都是疫情报告。所以还要利用正则表达式过滤一下，并把链接格式补完。

- 翻了几页，所有月度疫情公报都有“xx月全国法定”或“xx月份全国法定”的字样。
- 所有链接都是相对路径，形如../../xxx，需要替换为完整路径。

```r
library(stringr)
# 过滤出月报标题
toc <- toc[str_detect(toc$title, "月[份]*全国法定"),]
# 补完链接路径
toc$href <- str_replace(
    toc$href, "^.+/jkj(.+$)", "http://www.nhfpc.gov.cn/jkj\\1")
```

除了链接，我还想知道每份月报讲的是何年何月，将来解析了数据，直接可以对上日期。这时候标题就派上用场了，因为里面直接包含有年月信息。但这里就有一个坑：不是所有标题都完整。最早的一些月报是不含年份的，得结合发布日期来补全。

```r
# 哪些月报标题里没有“年”字？
idx.noyr <- which(! str_detect(toc$title, "\\d+年"))
# 生成date列，初始均为NA
toc$date <- NA
# 没有“年”字的月报，就用发布日期的年份来替代
toc$date[idx.noyr] <- paste(
    str_replace(toc$dop[idx.noyr], "(^\\d+).+", "\\1"),
    str_replace(toc$title[idx.noyr], ".+(\\d+)月.+", "\\1"), "1",
    sep="-")
# 其他月报，就直接提取标题里的年月信息
toc$date[-idx.noyr] <- str_replace(
    toc$title[-idx.noyr], "\\D*(\\d+)年(\\d+)月.+", "\\1-\\2-1")
# 默认都是当月1日。date列转为日期型
toc$date <- as.Date(toc$date)
```

`toc`数据集长这样：

![toc表格](http://ohghnje4x.bkt.clouddn.com/image/170404/toc_tbl.png)

万里长征踏出了第一步。

# 爬网页

这一步就比较简单了，干脆就先把网页代码先弄下来。在这里就不多考虑反爬虫问题了（因为试下来卫计委官网好像没有反爬虫机制，再说才爬它一百多个页面，有啥好反的）。电脑虽然配置不济，好歹有4个核。为了加点速，充分利用CPU（R默认只用单核）算力，不妨拿`doParallel`包做点并行计算处理。

```r
# 构造一个读取网页代码的函数
getWebPage <- function(url){
	## Arg
	##    url: 网页网址
	html <- read_lines(url)
	return(paste(html, collapse="\n"))
}
# 并行计算
library(doParallel)
registerDoParallel(cores=parallel::detectCores())
pages <- foreach(i=seq_along(toc$href), .combine=c) %dopar% 
	invisible(getWebPage(toc$href[i]))
names(pages) <- as.character(toc$date)
```

pages是一个149个文本元素构成的大向量，用日期作为向量命名。

```r
str(pages)
```

```r
 Named chr [1:149] "  <!DOCTYPE html PUBLIC \"-//W3C//DTD XHTML 1.0 Transitional//EN\" \"http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd\">"| __truncated__ ...
 - attr(*, "names")= chr [1:149] "2017-02-01" "2017-01-01" "2016-12-01" "2016-11-01" ...
```

# 解析发病和死亡总数

第二个大坑正在缓缓靠近：不是所有月报都有发病和死亡总数。

- 2005年以前，压根不报告丙类传染病
- 2010年以前，不汇报发病和死亡总数，且甲乙类合并计数，丙类另报
- 甲类汇总的文本多样化很强，一般鼠疫和霍乱还分开来报，正则写起来会死。

所以最后决定：

- 抽得到总数的，直接用总数
- 没有报告总数的，会向后抽到甲乙类合计，或甲类，或乙类。无论哪种情况，乙类+丙类基本等于总数

好机（偷）智（懒）！

```r
# 构造函数，从正文直接提取发病和死亡总数
getKeyNums <- function(page){
    ## Arg
    ##   page: 单个网页

    page <- read_html(page)
    txt.node <- html_nodes(page, "div.con")
    txt <- html_text(txt.node)  # 获得月报正文
    ## 发病总数
    inc.tot <- as.integer(str_replace(
        txt, regex(
            ".+?(发病|报告)\\D+(\\d{6,})[例人].+", dotall=TRUE
            ), "\\2"))
    ## 死亡总数
    mot.tot<- as.integer(str_replace(
        txt, regex(
            ".+?(发病|报告).+?死亡\\D*?(\\d+)[例人].+", dotall=TRUE
            ), "\\2"))
    ## 乙类发病总数
    inc.b <- as.integer(str_replace(
        txt, regex(
            ".+?乙类.+?(\\d+)[例人].+", dotall=TRUE
            ), "\\1"))
    ## 乙类死亡总数
    mot.b <- as.integer(str_replace(
        txt, regex(
            ".+?乙类.+?死亡\\D*?(\\d+)[例人].+", dotall=TRUE), 
        "\\1"))
    ## 丙类发病总数
    inc.c <- as.integer(str_replace(
        txt, regex(
            ".+?丙类.+?(\\d+)[例人].+", dotall=TRUE
            ), "\\1"))
    ## 丙类死亡总数
    mot.c <- as.integer(str_replace(
        txt, regex(
            ".+?丙类.+?死亡\\D*?(\\d+)[例人].+", dotall=TRUE
            ), "\\1"))
    ## 如发病总数=乙类发病数，则未提取到，用乙类+丙类替代
    if (identical(inc.tot, inc.b)) inc.tot <- inc.tot + inc.c
    if (identical(mot.tot, mot.b)) mot.tot <- mot.tot + mot.c
    ## 返回发病总数和死亡总数
    return(c(inc.tot, mot.tot))
}
# 遍历pages运行getKeyNums，转置后转换为2列的数据框
genl.stat <- as.data.frame(t(sapply(pages, getKeyNums)))
names(genl.stat) <- c("Incidence", "Mortality")
# 将row.names衍生为新变量Date，转为日期型
genl.stat$Date <- as.Date(row.names(genl.stat))
# 有重复月报！去掉重复记录。取2005年以后的月报
genl.stat <- genl.stat[
     ! duplicated(genl.stat$Date) & genl.stat$Date > "2004-12-01",]
genl.stat <- genl.stat[order(genl.stat$Date),]  # 按日期排序
# 2009/4的月报无论如何不适配，只能手动改
genl.stat["2009-04-01", 1:2] <- c(338281, 576)
```

历尽艰苦，终于得到了这个该死的数据集。把历月发病和死亡数做个时序图。

```r
genl.stat <- melt(genl.stat, id="Date")
library(ggplot2)
library(ggthemes)
ggplot(genl.stat) + geom_line(aes(Date, value, color=variable)) + theme_hc() +
    scale_color_hc() + scale_x_date(date_breaks="1 year", date_labels="%Y") +
    facet_grid(variable~., scales="free") + 
    theme(axis.ticks=element_line(linetype=0)) +
    labs(title="Incidence And Mortality of Notifiable Infectious Diseases",
        subtitle="2005/1-2017/2", caption="source: NHFPC")
```

年周期性还是很显著的。

![发病/死亡月度统计](http://ohghnje4x.bkt.clouddn.com/image/170404/inc_mot_infectdis.png)

# 抽取各明细病种数据

接下来进行明细分病种数据的爬取。

理论上，把表格抽取出来就完了嘛。但是尝试了一下发现竟然有问题。随机看了一下，发现自己被坑惨了。附表有三类：

- 网页表格，这种最好利用
- 附件，如doc、xls
- 图片（！），如png，jpg

竟然还有直接传个截图当公报的，怎么不去爆炸？

本来打算`html_table`跑一遍就愉快地合并数据框了，却不料跑进了一个马里亚纳深坑里。只能改变策略，先把这些附件都下载下来。

```r
# 构造函数，下载附表
getWebTbl <- function(url, tbl.name){
    ## Arg
    ##    url: 网页
    ##    tbl.name: 表格名称，用日期命名

    # 如附表文件已存在，跳出
    if (any(file.exists(
        paste0("~/infectdis/", tbl.name, ".", 
               c("xls", "csv", "doc", "gif", "jpg", "png"))))){
        return(invisible())
    }
	# 否则就把页面源码读下来
    html <- read_html(url)
    # 尝试抽取网页表格
    cast <- html_nodes(html, "table")
    # 尝试抽取附件表格
    cast.attach <- html_nodes(html, "div.con a")
    regex.attach <- "([Xx][Ll][Ss][Xx]*|[Dd][Oo][Cc][Xx]*)"
    # 尝试抽取附件图片
    cast.img <- html_nodes(html, "div.con img")
    regex.img <- "([Gg][Ii][Ff]|[Pp][Nn][Gg]|[Jj][Pp][Gg])"
    if (length(cast)>0){  # 读到网页表格
        out <- html_table(cast, fill=TRUE)[[1]]
        if (! file.exists(paste0("~/infectdis/", tbl.name, ".csv"))){
            write_csv(out, paste0("~/infectdis/", tbl.name, ".csv"))
        }
    } else if (any(str_detect(
        cast.attach, paste0("\\.", regex.attach, "\"")))){  
        # 读到附件表格
        idx.attach <- which(str_detect(
            cast.attach, paste0("\\.", regex.attach, "\"")))[1]
        doc.link <- str_replace(
            cast.attach[idx.attach], 
            paste0(".+href=\"(.+?\\.)", regex.attach, "\".+"), "\\1\\2")
        file.type <- tolower(str_replace(
            doc.link, paste0(".+\\.", regex.attach, "$"), "\\1"))
        # 附件路径补完
        if (str_detect(doc.link, "^/"))
            doc.link <- paste0(
                "http://www.nhfpc.gov.cn", doc.link)
        # 另一种类型的相对路径
        if (str_detect(doc.link, "^[^h/]"))
            doc.link <- paste0(
                str_replace(url, "^(.+)\\.shtml$", "\\1"),
                str_replace(doc.link, "^[^/]+(/.+$)", "\\1"))
        # 重命名并存储
        if (! file.exists(paste0(
            "~/infectdis/", tbl.name, ".", file.type))){
            doc.file <- download.file(
                doc.link, destfile=paste0(
                    "~/infectdis/", tbl.name, ".", file.type))
        }
    } else if (any(str_detect(cast.img, paste0(
        "\\.", regex.img, "\"")))){
        # 读到附件图片
        idx.img <- which(str_detect(
            cast.img, paste0("\\.", regex.img, "\"")))[1]
        doc.link <- str_replace(
            cast.img[idx.img], 
            paste0(".+img.+src=\"(.+?\\.)", regex.img, "\".+"), "\\1\\2")
        file.type <- tolower(str_replace(
            doc.link, paste0(".+\\.", regex.img, "$"), "\\1"))
        # 附件路径补完        
        if (str_detect(doc.link, "^/"))
            doc.link <- paste0(
                "http://www.nhfpc.gov.cn", doc.link)
        # 另一种类型的相对路径
        if (str_detect(doc.link, "^[^h/]"))
            doc.link <- paste0(
                str_replace(url, "^(.+)\\.shtml$", "\\1"),
                str_replace(doc.link, "^[^/]+(/.+$)", "\\1"))
        # 重命名并存储
        if (! file.exists(paste0(
            "~/infectdis/", tbl.name, ".", file.type))){
            doc.file <- download.file(
                doc.link, destfile=paste0(
                    "~/infectdis/", tbl.name, ".", file.type))
        }
    }
}
# 再次动用并行计算
registerDoParallel(cores=parallel::detectCores())
foreach(i=seq_along(toc$href)) %dopar% 
    invisible(getWebTbl(toc$href[i], as.character(toc$date[i])))
```

看到这些文件齐齐整整码在硬盘里，心下暂时宽慰了一点。

![附件下载](http://ohghnje4x.bkt.clouddn.com/image/170404/downfiles.png)

然而让我们回头看看疫情月报附表们可恨的多样性吧。

```r
table(str_replace(
    list.files("~/infectdis"), ".+\\.(.+$)", "\\1"))
```

```r
csv doc gif jpg png xls 
 67  51  21   4   1   2
```

140多篇动态报道，只有67篇用了网页表格附件，有53篇是MS office文档，还有26篇直接贴图了事。

真是一叶落而知天下秋。卫生系统的数据化水准，跟金融系统比起来真是天上地下，一个站着，一个躺着。

[待续]

----

<img src="http://ohghnje4x.bkt.clouddn.com/QRcode.jpg" width="50%">扫码关注我的的我的公众号</img>
