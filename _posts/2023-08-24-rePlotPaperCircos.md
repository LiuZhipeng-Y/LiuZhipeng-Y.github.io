---
layout: post
title: 文章图复现之circos
description: 复现最后的circos图。
author: Z.P.L.
categories: R
keywords: R code bioinfo Linux
---

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=520 height=52 src="//music.163.com/outchain/player?type=2&id=188703&auto=1&height=32"></iframe>

**10.1093/biolre/ioac216**

# 1 用到的R包

***rtracklayer***包没有就用**`BiocManager::install("rtracklayer")`**安装，包的三种安装方法（发布平台）：CRAN、github、Bioconductor.
对应三种安装方法

- `install.packages()`

- `remotes::install_github()`

- `BiocManager::install()`

首先谷歌搜索未安装的包是发布在哪个平台，一般是CRAN和Bioconductor，github大多是一些个人开发的包或最新版（不稳定版）会先发布到github.

``` r
library(circlize)#画circos的包
library(rtracklayer)
library(stringr)
library(tidyverse)
library(grid)
library(scales)
library(ggsci)
```

# 2 数据准备⭐

## 2.1 首先获得noncoding RNA ID

读入预测编码能力的文件，获得非编码RNA的ID

``` r
abun2 <- read_delim("E:/gder/Tes_Epi.fl.count.tsv",col_names = T,delim = "\t")
cpc2 <- read_delim("E:/gder/coding_predict_data/CPC2.txt.txt",delim = "\t",col_names = T,col_select = c(pbid=1,type=8)) %>% 
  inner_join(.,abun2,by="pbid") %>% 
  filter(type=="noncoding")
cnci <- read_delim("E:/gder/coding_predict_data/CNCI.index",delim = "\t",col_names = T,col_select = c(pbid=1,type=2)) %>% 
  inner_join(.,abun2,by="pbid") %>% 
  filter(type=="noncoding")
cpat <- read_delim("E:/gder/coding_predict_data/cpat.ORF_prob.best.tsv",delim = "\t",col_names = T,col_select = c(pbid=1,prob=11)) %>% 
  dplyr::mutate(type=ifelse(prob>0.71,"coding","noncoding")) %>% dplyr::select(1,3) %>% 
  inner_join(.,abun2,by="pbid") %>% 
  filter(type=="noncoding")
pfam <- read_table("E:/gder/coding_predict_data/pfam.all.txt",skip = 29,col_names = F) %>% 
  dplyr::mutate(type=ifelse(X13>0.001,"noncoding","coding")) %>%
  filter(type=="coding") %>% 
  dplyr::select(pbid="X1",type) %>% 
  distinct() %>% 
  full_join(.,abun2,by="pbid") %>% 
  filter(is.na(type))

#提取非编码的id
gplots::venn(list(CNCI=cnci[cnci$FG.count>0,]$pbid,CPC2=cpc2[cpc2$FG.count>0,]$pbid,Pfam=pfam[pfam$FG.count>0,]$pbid,CPAT=cpat[cpat$FG.count>0,]$pbid),show.plot = F) -> noncod.venn.fg
gplots::venn(list(CNCI=cnci[cnci$GW.count>0,]$pbid,CPC2=cpc2[cpc2$GW.count>0,]$pbid,Pfam=pfam[pfam$GW.count>0,]$pbid,CPAT=cpat[cpat$GW.count>0,]$pbid),show.plot = F) -> noncod.venn.gw
attributes(noncod.venn.fg)$intersections$`CNCI:CPC2:Pfam:CPAT` -> noncod.fg
attributes(noncod.venn.gw)$intersections$`CNCI:CPC2:Pfam:CPAT` -> noncod.gw
```

## 2.2 读取第一圈数据

读入第一圈数据 **I**; cytoband.df为染色体长度信息

``` r
cytoband.df <- read.table("E:/gder/Fig6.circos/cytoBandIdeo.txt", colClasses = c("character", "numeric",
                                                                                         "numeric", "character", "character"), sep = "\t")
head(cytoband.df)
```

    ##      V1 V2        V3 V4   V5
    ## 1  chr1  0 274330532    gneg
    ## 2 chr10  0  69359453    gneg
    ## 3 chr11  0  79169978    gneg
    ## 4 chr12  0  61602749    gneg
    ## 5 chr13  0 208334590    gneg
    ## 6 chr14  0 141755446    gneg

## 2.3 准备第II和IV圈数据

读入三代获得的注释信息，准备 **II** 和 **IV**
圈数据，GW和FG鉴定到的isoform，

使用**rtracklayer**包中的`import()`函数导入**gtf**文件

AEMK和MT的染色体删除，只保留主要的染色体

``` r
sqanti3_class <- read_delim("E:/gder/SQANTI3.mergefinal_classification.txt",col_names = T,delim = "\t")
isoseqgtf <- rtracklayer::import("E:/gder/Fig6.circos/SQANTI3.mergefinal_corrected.gtf")
isoseqgtf <- as.data.frame(isoseqgtf)
isoseqgtf <- isoseqgtf[-grep("AEMK",isoseqgtf$seqnames),]#去除AEMK染色体
isoseqgtf <- isoseqgtf[-grep("MT",isoseqgtf$seqnames),]#去除MT染色体
isoseqgtf$seqnames <- paste("chr",isoseqgtf$seqnames,sep = "")

isoseqgtf %>% 
  filter(type=="transcript") %>% 
  inner_join(.,abun2,by=c("transcript_id"="pbid")) %>% 
  dplyr::select(seqnames,start,end,transcript_id,GW.count) %>% 
  filter(GW.count>0) -> gw.isoform.bed

isoseqgtf %>% 
  filter(type=="transcript") %>% 
  inner_join(.,abun2,by=c("transcript_id"="pbid")) %>% 
  dplyr::select(seqnames,start,end,transcript_id,FG.count) %>% 
  filter(FG.count>0) -> fg.isoform.bed
head(fg.isoform.bed)
```

    ##   seqnames  start    end transcript_id FG.count
    ## 1     chr1  23811  40028        PB.1.1       14
    ## 2     chr1 121145 186749        PB.2.1        5
    ## 3     chr1 121146 186752        PB.2.2       14
    ## 4     chr1 121404 186759        PB.2.3        4
    ## 5     chr1 169865 186757        PB.2.4        4
    ## 6     chr1 575725 578418        PB.3.1        4

## 2.4 准备第III和V圈数据

准备 **III** 和 **V** 圈数据，GW和FG预测到的非编码RNA

第一步获得的非编码RNA的id是字符串，这里要用`tibble()`或`as.data.frame()`转换为数据框才能合并

``` r
isoseqgtf %>% 
  filter(type=="transcript") %>% 
  inner_join(.,tibble(pbid=noncod.gw),by=c("transcript_id"="pbid")) %>% 
  dplyr::select(seqnames,start,end,transcript_id) %>% 
  inner_join(.,sqanti3_class[,c(1,4)],by=c("transcript_id"="isoform")) -> gw.non.bed
isoseqgtf %>% 
  filter(type=="transcript") %>% 
  inner_join(.,tibble(pbid=noncod.fg),by=c("transcript_id"="pbid")) %>% 
  dplyr::select(seqnames,start,end,transcript_id) %>% 
  inner_join(.,sqanti3_class[,c(1,4)],by=c("transcript_id"="isoform")) -> fg.non.bed
head(fg.non.bed)
```

    ##   seqnames    start      end transcript_id length
    ## 1     chr1  6730403  6733062       PB.10.1   2660
    ## 2     chr1 57651685 57654846       PB.77.1   1075
    ## 3     chr1 62590470 62590714       PB.78.1    245
    ## 4     chr1 73119761 73124353       PB.90.2   1835
    ## 5     chr1 73121192 73124353       PB.90.3    404
    ## 6     chr1 74757545 74760323       PB.94.1   2779

## 2.5 准备第VI圈数据

***Ensembl*** 数据，参考注释文件，第 ***VI*** 圈

加载较慢，需要等待；数据较大，建议`head()`查看

``` r
ensegtf <- rtracklayer::import("E:/gder/Fig6.circos/Sus_scrofa.Sscrofa11.1.107.gtf.gz")
ensegtf <- as.data.frame(ensegtf)
ensegtf <- ensegtf[-grep("AEMK",ensegtf$seqnames),]
ensegtf <- ensegtf[-grep("MT",ensegtf$seqnames),]
ensegtf$seqnames <- paste0("chr",ensegtf$seqnames)
ensegtf.trans <- ensegtf[ensegtf$type=="transcript",]
ensegtf.gene <- ensegtf[ensegtf$type=="gene",]
head(ensegtf.gene)
```

    ##     seqnames     start       end  width strand  source type score phase
    ## 1       chr1 226161299 226217308  56010      - ensembl gene    NA    NA
    ## 187     chr1 226414306 226432279  17974      + ensembl gene    NA    NA
    ## 248     chr1 227574648 227772671 198024      + ensembl gene    NA    NA
    ## 351     chr1 227811204 227963309 152106      - ensembl gene    NA    NA
    ## 545     chr1 228033996 228045013  11018      - ensembl gene    NA    NA
    ## 557     chr1 228054915 228092263  37349      - ensembl gene    NA    NA
    ##                gene_id gene_version gene_name gene_source   gene_biotype
    ## 1   ENSSSCG00000028996            4   ALDH1A1     ensembl protein_coding
    ## 187 ENSSSCG00000005267            6     ANXA1     ensembl protein_coding
    ## 248 ENSSSCG00000005268            5      RORB     ensembl protein_coding
    ## 351 ENSSSCG00000005269            5     TRPM6     ensembl protein_coding
    ## 545 ENSSSCG00000031382            2   C9orf40     ensembl protein_coding
    ## 557 ENSSSCG00000005271            5   CARNMT1     ensembl protein_coding
    ##     transcript_id transcript_version transcript_name transcript_source
    ## 1            <NA>               <NA>            <NA>              <NA>
    ## 187          <NA>               <NA>            <NA>              <NA>
    ## 248          <NA>               <NA>            <NA>              <NA>
    ## 351          <NA>               <NA>            <NA>              <NA>
    ## 545          <NA>               <NA>            <NA>              <NA>
    ## 557          <NA>               <NA>            <NA>              <NA>
    ##     transcript_biotype exon_number exon_id exon_version protein_id
    ## 1                 <NA>        <NA>    <NA>         <NA>       <NA>
    ## 187               <NA>        <NA>    <NA>         <NA>       <NA>
    ## 248               <NA>        <NA>    <NA>         <NA>       <NA>
    ## 351               <NA>        <NA>    <NA>         <NA>       <NA>
    ## 545               <NA>        <NA>    <NA>         <NA>       <NA>
    ## 557               <NA>        <NA>    <NA>         <NA>       <NA>
    ##     protein_version projection_parent_transcript  tag
    ## 1              <NA>                         <NA> <NA>
    ## 187            <NA>                         <NA> <NA>
    ## 248            <NA>                         <NA> <NA>
    ## 351            <NA>                         <NA> <NA>
    ## 545            <NA>                         <NA> <NA>
    ## 557            <NA>                         <NA> <NA>

## 2.6 准备第VII圈数据

第 ***VII*** 圈，融合基因

``` r
isoseqgtf %>% 
  filter(type=="transcript") %>% 
  inner_join(.,abun2,by=c("transcript_id"="pbid")) %>% 
  dplyr::select(seqnames,start,end,transcript_id,GW.count,FG.count) %>% 
  inner_join(.,sqanti3_class[,c(1,6)],by=c("transcript_id"="isoform")) %>% 
  filter(structural_category=="fusion") %>% 
  dplyr::select(1:3,5:6) %>% 
  pivot_longer(cols = 4:5) %>% 
  filter(value>0) %>% 
  mutate(va=ifelse(name=="GW.count",1,1.01)) %>% 
  dplyr::select(1:3,6) -> fusion.bed
head(fusion.bed)
```

    ## # A tibble: 6 × 4
    ##   seqnames     start       end    va
    ##   <chr>        <int>     <int> <dbl>
    ## 1 chr1      28968843  29052422  1   
    ## 2 chr1      28968843  29052422  1.01
    ## 3 chr1     136704410 137163550  1   
    ## 4 chr1     136704412 137163550  1   
    ## 5 chr1     136916922 137163561  1   
    ## 6 chr1     137998583 138191950  1

## 2.7 添加猪的图片

``` r
#pigimg <- png::readPNG("E:/gder/Bagua-name-earlier.png")
pigimg <- png::readPNG("E:/gder/Bagua-name-earlier.png")
```

# 3 开始绘图

❗这只是一个模板

官方文档参考 <https://jokergoo.github.io/circlize_book/book>

***一步一步运行***

## 3.1 I; 画染色体

出现Note: 1 point is out of plotting region in sector ‘chr5’, track
‘1’.并不是报错，只是把标签加到了圈外，把`+ mm_y(3.8)`删除就不会出现此note

``` r
circos.clear()#清除画板
circos.par(start.degree = 83, gap.after = c(rep(1, 19), 15),cell.padding = c(0, 0, 0, 0))#建立画板
circos.initializeWithIdeogram(cytoband.df,plotType = NULL)#初始化
circos.genomicTrackPlotRegion(
  cytoband.df, track.height = 0.04, stack = TRUE, bg.border="black",
  panel.fun = function(region, value, ...) {
    chr=CELL_META$sector.index
    xlim=CELL_META$xlim
    ylim=CELL_META$ylim
    circos.text(CELL_META$xcenter, CELL_META$ylim[2] + mm_y(3.8), chr, cex = 1, col = "black",
                niceFacing = TRUE)
  } ,bg.col = "#ccccd6")
```

<img src="/images/posts/circos/unnamed-chunk-10-1.png" style="display: block; margin: auto;" />

**添加染色体尺度**并添加**I**标签

``` r
#染色体尺度
brk <- c(0,50,100,150,200,250,300)*1000000
#brk_label<-paste0(c(0,50,100,150,200,250,300),"M")
brk_label<-c(0,50,100,150,200,250,300)
circos.track(track.index = get.current.track.index(), 
             panel.fun = function(x, y) {
               circos.axis(h="top",
                           major.at=brk,
                           labels=brk_label,
                           labels.cex=0.55,
                           col="black",
                           labels.col="black",
                           major.tick.length = mm_y(0.5),
                           labels.facing="clockwise",)
             },
             bg.border=F)
circos.text(x = CELL_META$xlim[2] + mm_x(5),y = CELL_META$ycenter,labels = "I",
            sector.index = "chrY",track.index = 1,cex = 1.5,col = "black")
```

<img src="/images/posts/circos/unnamed-chunk-11-1.png" style="display: block; margin: auto;" />

## 3.2 II; GW isoform density

添加第二圈

``` r
## II
circos.genomicDensity(gw.isoform.bed[,1:3],col = c("#F08650"), track.height = 0.1,
                      bg.border = NA,bg.col = NA ,type = "h",overlap = F,count_by = "number",
                      window.size = 1e5)
circos.text(x = CELL_META$xlim[2] + mm_x(4.5),y = CELL_META$ycenter,labels = "II",
            sector.index = "chrY",track.index = 2,cex = 1.5,col = "black")
```

<img src="/images/posts/circos/unnamed-chunk-12-1.png" style="display: block; margin: auto;" />

## 3.3 III; GW noncod point

添加第三圈

``` r
# III; GW noncod point
max(fg.non.bed$length,gw.non.bed$length) -> MAX
circos.trackPlotRegion(ylim = c(0, 7000),track.height = 0.08,bg.col = "#f0e9e6", bg.border = NA,
                       panel.fun = function(x,y) {
                         for (i in seq(0,7000,1000)) {
                           circos.lines(c(0,CELL_META$xlim[2]),c(i,i),col = "#939393",lwd = 0.1)
                         }
                       })
for (chromosome in unique(gw.non.bed$seqnames)){
  circos.points(sector.index = chromosome,
                 x = gw.non.bed[gw.non.bed$seqnames==chromosome,]$start,
                 y = gw.non.bed[gw.non.bed$seqnames==chromosome,]$length,
                 pch = 16,
                 col = "#F08650",
                 cex = 0.3,
                 bg=NA)
}
circos.text(x = CELL_META$xlim[2] + mm_x(4),y = CELL_META$ycenter,labels = "III",
            sector.index = "chrY",track.index = 3,cex = 1.5,col = "black")
```

<img src="/images/posts/circos/unnamed-chunk-13-1.png" style="display: block; margin: auto;" />

## 3.4 IV; FG isoform density

添加第**四**圈

``` r
## IV; FG isoform density
circos.genomicDensity(fg.isoform.bed[,1:3],col = c("#9adafa"), track.height = 0.1,
                      bg.border = NA,bg.col = NA ,type = "h",overlap = F,count_by = "number",
                      window.size = 1e5)
circos.text(x = CELL_META$xlim[2] + mm_x(3.5),y = CELL_META$ycenter ,labels = "IV",
            sector.index = "chrY",track.index = 4,cex = 1.5,col = "black")
```

<img src="/images/posts/circos/unnamed-chunk-14-1.png" style="display: block; margin: auto;" />

## 3.5 V; FG noncod point

添加第**五**圈

``` r
# V; FG noncod point
circos.trackPlotRegion(ylim = c(0, 7000),track.height = 0.08,bg.col = "#e9f3fa", bg.border = NA,
                       panel.fun = function(x,y) {
                         for (i in seq(0,7000,1000)) {
                           circos.lines(c(0,CELL_META$xlim[2]),c(i,i),col = "#939393",lwd = 0.1)
                         }
                       })
for (chromosome in unique(fg.non.bed$seqnames)){
  circos.points(sector.index = chromosome,
                x = fg.non.bed[fg.non.bed$seqnames==chromosome,]$start,
                y = fg.non.bed[fg.non.bed$seqnames==chromosome,]$length,
                pch = 16,
                col = "#9adafa",
                cex = 0.3,
                bg=NA)
}
circos.text(x = CELL_META$xlim[2] + mm_x(3),y = CELL_META$ycenter,labels = "V",
            sector.index = "chrY",track.index = 5,cex = 1.5,col = "black")
```

<img src="/images/posts/circos/unnamed-chunk-15-1.png" style="display: block; margin: auto;" />

## 3.6 VI; Ensembl isoform density

添加第**六**圈

``` r
## VI; Ensembl isoform density
circos.genomicDensity(ensegtf.trans[,1:3],col = c("#8ed3c9"), track.height = 0.1,
                      bg.border = NA,bg.col = NA ,type = "h",overlap = F,count_by = "number",
                      window.size = 1e5)
circos.text(x = CELL_META$xlim[2] + mm_x(2.5),y = CELL_META$ycenter,labels = "VI",
            sector.index = "chrY",track.index = 6,cex = 1.5,col = "black")
```

<img src="/images/posts/circos/unnamed-chunk-16-1.png" style="display: block; margin: auto;" />

## 3.7 VII; fusion

添加第**七**圈

``` r
# VII; fusion
circos.trackPlotRegion(ylim = c(0, 1.01),track.height = 0.08,bg.col = NA, bg.border = NA,
                       panel.fun = function(x,y) {
                           circos.lines(c(0,CELL_META$xlim[2]),c(0,0),col = "#D1D1D1",lwd = 0.1)
                       })
for (chromosome in unique(fusion.bed$seqnames)){
  circos.barplot(sector.index = chromosome,
                 value = fusion.bed[fusion.bed$seqnames==chromosome,]$va,
                 pos = fusion.bed[fusion.bed$seqnames==chromosome,]$start,
                 col = ifelse(fusion.bed[fusion.bed$seqnames==chromosome,]$va==1,"#9adafa","#F08650"),
                 bar_width = 3500000,
                 border=NA)
}
circos.text(x = CELL_META$xlim[2] + mm_x(2),y = CELL_META$ycenter,labels = "VII",
            sector.index = "chrY",track.index = 7,cex = 1.5,col = "black")
grid.raster(pigimg,x = 0.5,y = 0.5,width = 0.12,height = 0.12)#加个图
```

<img src="/images/posts/circos/unnamed-chunk-17-1.png" style="display: block; margin: auto;" />

# 4 Done⭐
