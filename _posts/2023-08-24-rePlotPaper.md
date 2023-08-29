---
layout: post
title: 文章图复现
description: 复现一篇文章中的所有图，不包括手工绘制的图。
author: Z.P.L.
categories: R
keywords: R code bioinfo Linux
---

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=520 height=52 src="//music.163.com/outchain/player?type=2&id=190473&auto=1&height=32"></iframe>

**10.1093/biolre/ioac216**

👋**这是Fig2-5的代码，可以复制**

------------------------------------------------------------------------

# 1 Figure2

## 1.1 用到的R包

``` r
library(scales)
library(dplyr)
library(cowplot)
library(ggpubr)
library(ggthemes)
library(tidyverse)
```

## 1.2 画Fig2B

``` r
GW <- read_delim("E:/gder/JJ27GW.fl.len.txt",col_names = c("id","len"),delim = "\t")
FG <- read_delim("E:/gder/JJ27FG.fl.len.txt",col_names = c("id","len"),delim = "\t")

GW %>% 
  ggplot(aes(len)) +
  geom_histogram(fill="#9fd4f4",color="black") +
  scale_x_log10(breaks = trans_breaks("log10", function(x) 10^x),
                labels = trans_format("log10", math_format(10^.x))) -> p
#查看y轴的范围
ggplot_build(p)$layout$panel_scales_y -> RANGE

#将累积曲线的y轴放大
GW %>% 
  ggplot(aes(x = len))+
  geom_histogram(fill="#9fd4f4",color="black") +
  ##Y轴放大
  geom_line(data = data.frame(cumsum(table(GW$len))/length(GW$len)), aes(x = as.numeric(names(table(GW$len))), 
                                                                           y = cumsum(table(GW$len))/length(GW$len)*max(RANGE[[1]]$range$range)), color = "#6d91a7") +
  scale_y_continuous(name = "Count", sec.axis = sec_axis(trans = ~./max(RANGE[[1]]$range$range), name = "Cumulative Percentage (%)"),limits = c(0,41000)) +
  scale_x_log10(name = "Length (nt)",breaks = trans_breaks("log10",function(x) 10^x),
                labels = trans_format("log10",math_format(10^.x))) +
  theme_classic() -> fig2b1

FG %>% 
  ggplot(aes(len)) +
  geom_histogram(fill="#9fd4f4",color="black") +
  scale_x_log10(breaks = trans_breaks("log10", function(x) 10^x),
                labels = trans_format("log10", math_format(10^.x))) -> p
#查看y轴的范围
ggplot_build(p)$layout$panel_scales_y -> RANGE

#将累积曲线的y轴放大
FG %>% 
  ggplot(aes(x = len))+
  geom_histogram(fill="#ecc1dd",color="black") +
  ##Y轴放大
  geom_line(data = data.frame(cumsum(table(FG$len))/length(FG$len)), aes(x = as.numeric(names(table(FG$len))), 
                                                                         y = cumsum(table(FG$len))/length(FG$len)*max(RANGE[[1]]$range$range)), color = "#ecc1dd") +
  scale_y_continuous(name = "Count", sec.axis = sec_axis(trans = ~./max(RANGE[[1]]$range$range), name = "Cumulative Percentage (%)"),limits = c(0,41000)) +
  scale_x_log10(name = "Length (nt)",breaks = trans_breaks("log10",function(x) 10^x),
                labels = trans_format("log10",math_format(10^.x))) +
  theme_classic() -> fig2b2
plot_grid(fig2b1,fig2b2,align = "hv",axis = "lr",nrow = 1) ##组合图形
```

![](/images/posts/replotBOR/unnamed-chunk-2-1.png)<!-- -->

## 1.3 画Fig2C

### 1.3.1 boxplot

``` r
GW <- read_delim("E:/gder/JJ27GW.fl.len.txt",col_names = c("id","len"),delim = "\t")
FG <- read_delim("E:/gder/JJ27FG.fl.len.txt",col_names = c("id","len"),delim = "\t")
ense <- read_delim("E:/gder/trans.len.txt",col_names = T,delim = "\t")
names(ense) <- c("id","len")
GW$type <- "Testis"  #新增名为type的列，内容为“Testis”
FG$type <- "Epididymis"
ense$type <- "Ensembl"
rbind(GW,FG,ense) -> len.box.dat
len.box.dat %>% 
  mutate(type=factor(type,levels=c("Testis","Epididymis","Ensembl"))) %>% #type列里的值排序
  ggplot(aes(x=type,y=len,fill=type))+ ##fill放在aes(),用于颜色映射
  geom_boxplot(outlier.shape = NA,notch = T,linetype="dashed")+ ##误差棒以外的点设为NA,意思为不显示这些点
  stat_boxplot(aes(ymin = ..lower.., ymax = ..upper..), outlier.shape = NA) + ##添加误差棒
  stat_boxplot(geom = "errorbar", aes(ymin = ..ymax..),width=0.5) +
  stat_boxplot(geom = "errorbar", aes(ymax = ..ymin..),width=0.5) +
  scale_fill_manual(values = c("#9486aa","#f18eba","#a3d198"))+ 
  scale_y_log10(breaks = trans_breaks("log10",function(x) 10^x), #y轴的刻度表现形式
                labels = trans_format("log10",math_format(10^.x))) +
  coord_cartesian(ylim = c(300,20000)) +  ##截取Y轴区间
  labs(x=NULL,y="Length (nt)")+
  theme_hc() +
  theme(legend.position = "none")
```

![](/images/posts/replotBOR/unnamed-chunk-3-1.png)<!-- -->

### 1.3.2 柱形图

``` r
GW <- read_delim("E:/gder/JJ27GW.clustered.len.txt",col_names = c("id","len"),delim = "\t")
FG <- read_delim("E:/gder/JJ27FG.clustered.len.txt",col_names = c("id","len"),delim = "\t")
ense <- read_delim("E:/gder/trans.len.txt",col_names = c("id","len"),delim = "\t",skip = 1)

GW$type <- "Testis"  #新增名为type的列，内容为“Testis”
FG$type <- "Epididymis"
ense$type <- "Ensembl"
rbind(GW,FG,ense) -> len.bar.dat
len.bar.dat$log10t <- log10(len.bar.dat$len)
MAX <- max(len.bar.dat$log10t)
diff_breaks <- c(0,seq(1.6, 4.2, by = 0.2),MAX)
len.bar.dat$Cat = cut(len.bar.dat$log10t, breaks = diff_breaks) ##生成长度区间
head(len.bar.dat)##查看len.bar.dat数据结构
```

    ## # A tibble: 6 × 5
    ##   id             len type   log10t Cat    
    ##   <chr>        <dbl> <chr>   <dbl> <fct>  
    ## 1 transcript/0 12395 Testis   4.09 (4,4.2]
    ## 2 transcript/1 10423 Testis   4.02 (4,4.2]
    ## 3 transcript/2  9833 Testis   3.99 (3.8,4]
    ## 4 transcript/3  9058 Testis   3.96 (3.8,4]
    ## 5 transcript/4  8998 Testis   3.95 (3.8,4]
    ## 6 transcript/5  8656 Testis   3.94 (3.8,4]

``` r
len.bar.dat %>% 
  group_by(type) %>% #按type进行分组
  dplyr::count(Cat) %>% #分组后计算每个type里Cat的个数
  mutate(type=factor(type,levels=c("Testis","Epididymis","Ensembl"))) %>%
  ggplot(aes(x=Cat,y=n,fill=type))+
  geom_bar(stat = "identity",position=position_dodge(0.9),color="black")+
  scale_fill_manual(values = c("#9486aa","#f18eba","#a3d198"))+
  scale_y_log10(name="Count",breaks = trans_breaks("log10",function(x) 10^x), #y轴的刻度表现形式
                labels = trans_format("log10",math_format(10^.x))) +
  theme_classic()+
  xlab("Length Intervals (Log10 nt)")+
  theme(legend.position = "none") -> fig2c2.1
##柱子宽度不一致，可以看出是有些数据缺失了
len.bar.dat %>% 
  group_by(type) %>% 
  dplyr::count(Cat) %>% 
  ungroup() %>% 
  add_row(type="Testis",Cat="(1.6,1.8]",n=1) %>% #加上缺失的行
  add_row(type="Epididymis",Cat="(1.6,1.8]",n=1) %>% 
  add_row(type="Epididymis",Cat="(4,4.2]",n=1) %>% 
  add_row(type="Testis",Cat="(4.2,4.41]",n=1) %>% 
  add_row(type="Epididymis",Cat="(4.2,4.41]",n=1) %>% 
  mutate(type=factor(type,levels=c("Testis","Epididymis","Ensembl"))) %>%
  ggplot(aes(x=Cat,y=n,fill=type))+
  geom_bar(stat = "identity",position=position_dodge(0.9),color="black") +
  scale_fill_manual(values = c("#9486aa","#f18eba","#a3d198"))+
  scale_y_log10(name="Count",breaks = trans_breaks("log10",function(x) 10^x), #y轴的刻度表现形式
                labels = trans_format("log10",math_format(10^.x))) +
  theme_classic()+
  xlab("Length Intervals (Log10 nt)")+
  theme(legend.position = "none") -> fig2c2.2
##组合p.sim1和p.sim2, cowplot::表示用cowplot包里的某个函数
cowplot::plot_grid(fig2c2.1,fig2c2.2,nrow = 1,labels = c("before","after"))
```

![](/images/posts/replotBOR/unnamed-chunk-4-1.png)<!-- -->

------------------------------------------------------------------------

# 2 Figure3

Figure3A用AI(Adobe Illustrator)画

## 2.1 Fig3B

``` r
abun2 <- read_delim("E:/gder/Tes_Epi.fl.count.tsv",col_names = T,delim = "\t")
sqanti3_class <- read_delim("E:/gder/SQANTI3.mergefinal_classification.txt",col_names = T,delim = "\t")
#查看每次操作后产生的数据类型，列名，才能进行下一步操作
sqanti3_class %>% 
  inner_join(.,abun2,by=c("isoform"="pbid")) %>% #按列合并两个数据框，类似于merge()函数
  dplyr::select(structural_category,GW.count,FG.count) %>% 
  tidyr::pivot_longer(cols = 2:3,names_to = "type") %>% #转换成长数据，自行查看pivot_longer()的用法
  filter(value>0) %>% 
  group_by(type) %>% 
  dplyr::count(structural_category) %>% 
  pivot_wider(names_from = "type",values_from = "n") %>% #转换成宽数据，自行查看pivot_wider()的用法
  dplyr::mutate(structural_category=factor(structural_category,levels=c("full-splice_match","incomplete-splice_match","novel_in_catalog", 
                                                                        "novel_not_in_catalog","antisense", "fusion", "intergenic","genic"))) %>% #转换为因子，方便排序
  dplyr::arrange(structural_category) -> class.result#排序
##数据和文章里的有些出入，但不影响
class.result
```

    ## # A tibble: 8 × 3
    ##   structural_category     FG.count GW.count
    ##   <fct>                      <int>    <int>
    ## 1 full-splice_match           1819     1894
    ## 2 incomplete-splice_match     1657     1639
    ## 3 novel_in_catalog            1938     1960
    ## 4 novel_not_in_catalog        2322     2801
    ## 5 antisense                     76       61
    ## 6 fusion                       100      115
    ## 7 intergenic                   123      190
    ## 8 genic                        138      160

## 2.2 Fig3C

``` r
#install.package("VennDiagram")#没有安装就执行这条命令
library(VennDiagram)
#看看venn.diagram()的用法
sqanti3_class %>% 
  inner_join(.,abun2,by=c("isoform"="pbid")) %>% #按列合并两个数据框，类似于merge()函数
  dplyr::select(associated_gene,GW.count,FG.count) %>% 
  tidyr::pivot_longer(cols = 2:3,names_to = "type") %>%
  filter(value>0) %>% 
  select(associated_gene,type) %>% 
  distinct() -> venn.gene
venn.diagram(x = list(Testis=venn.gene[venn.gene$type=="GW.count",]$associated_gene,Epididymis=venn.gene[venn.gene$type=="FG.count",]$associated_gene),
             filename = NULL,#输出文件名NULL
             fontfamily="sans",cat.fontfamily="sans",#设置字体
             fill=c("#bae0fa","#f18eba"),#填充色
             col="black",#线条颜色
             lwd=0.1,#线条粗细
             margin=0.1,#周边区域大小
             height=3,#图的宽高
             width=3) -> venn.gene.p
#用这个函数也可以画grid.draw(venn.iso)
#但推荐用下面的函数
cowplot::plot_grid(venn.gene.p)
```

![](/images/posts/replotBOR/unnamed-chunk-6-1.png)<!-- -->

## 2.3 Fig3D

``` r
#isoform per gene  
#为什么使用dplyr::的形式来调用函数，因为有些函数的名字是重复的，但是功能不一样，此时就要指定包名来调用函数
sqanti3_class %>% 
  inner_join(.,abun2,by=c("isoform"="pbid")) %>% 
  dplyr::select(isoform,associated_gene,GW.count,FG.count) %>% 
  tidyr::pivot_longer(cols = 3:4,names_to = "type",values_to = "counts") %>% 
  dplyr::filter(counts > 0) %>% 
  dplyr::group_by(type) %>% 
  dplyr::count(associated_gene,name = "iso.num") %>% 
  dplyr::count(iso.num,name = "gene.num") %>% 
  dplyr::mutate(isoper=ifelse(iso.num >6,"6+",iso.num)) %>% 
  dplyr::group_by(type,isoper) %>% 
  dplyr::summarise(Counts=sum(gene.num)) %>% 
  ungroup() %>% 
  dplyr::mutate(sample=ifelse(type=="GW.count","Testis","Epididymis")) %>% 
  dplyr::mutate(sample=factor(sample,levels=c("Testis","Epididymis"))) %>% 
  ggplot(aes(x=isoper,y=Counts,fill=sample))+
  geom_bar(stat="identity",position = position_dodge(0.9),color="black")+
  scale_fill_manual(values = c("#9486aa","#f18eba"))+
  scale_y_continuous(breaks = seq(0,3500,500))+
  theme_classic()+
  xlab("Number of isoforms per gene") +
  theme(legend.position = c(.9,.9),legend.title = element_blank()) -> p.fig3d
#组合韦恩图和柱状图
cowplot::plot_grid(p.fig3d)
```

![](/images/posts/replotBOR/unnamed-chunk-7-1.png)<!-- -->

------------------------------------------------------------------------

# 3 Figure4A

## 3.1 差异分析

``` r
#install.packages("BiocManager")#没有就安装
#BiocManager::install("DESeq2")#安装较慢，需要安装很多依赖包
library(DESeq2)
library(ggrepel)
library(ggplot2)
#差异分析，差异分析结果可以保存，分析的差异基因可用于GO和KEGG通路富集分析
geneinfo <- read_delim("E:/gder/ssc.geneinfo.ens107.txt",col_names = T,delim = "\t")
count <- read.table("E:/gder/All.featureCounts.txt",row.names = 1,header = T,sep = "\t")
count <- count[rowMeans(count)>10,]
head(count)
```

    ##                    BMI_Epididymis_RNAseq_rep1 BMI_Epididymis_RNAseq_rep2
    ## ENSSSCG00000028996                      18565                      21476
    ## ENSSSCG00000005267                       4976                       2513
    ## ENSSSCG00000005268                         22                         18
    ## ENSSSCG00000005269                         11                          8
    ## ENSSSCG00000031382                        186                        182
    ## ENSSSCG00000005271                       1559                       1149
    ##                    BMI_Epididymis_RNAseq_rep3 BMI_Epididymis_RNAseq_rep4
    ## ENSSSCG00000028996                      43210                      21565
    ## ENSSSCG00000005267                       3350                       3148
    ## ENSSSCG00000005268                         21                         13
    ## ENSSSCG00000005269                         35                          9
    ## ENSSSCG00000031382                        288                        200
    ## ENSSSCG00000005271                       1351                       2666
    ##                    BMI_Epididymis_RNAseq_rep5 BMI_Testis_RNAseq_rep1
    ## ENSSSCG00000028996                      22889                    297
    ## ENSSSCG00000005267                       3469                    136
    ## ENSSSCG00000005268                         17                     23
    ## ENSSSCG00000005269                         16                     33
    ## ENSSSCG00000031382                        261                    871
    ## ENSSSCG00000005271                       3014                     79
    ##                    BMI_Testis_RNAseq_rep2 BMI_Testis_RNAseq_rep3
    ## ENSSSCG00000028996                    225                    172
    ## ENSSSCG00000005267                    265                    145
    ## ENSSSCG00000005268                     46                     44
    ## ENSSSCG00000005269                     66                     34
    ## ENSSSCG00000031382                   1041                    833
    ## ENSSSCG00000005271                    150                     73
    ##                    BMI_Testis_RNAseq_rep4 BMI_Testis_RNAseq_rep5
    ## ENSSSCG00000028996                    383                    306
    ## ENSSSCG00000005267                    425                    184
    ## ENSSSCG00000005268                     34                     38
    ## ENSSSCG00000005269                     25                     66
    ## ENSSSCG00000031382                    665                   1077
    ## ENSSSCG00000005271                    171                    122

``` r
sa <- data.frame(ind=colnames(count),gr=str_split(colnames(count),"[_]",simplify = T)[,2])#制作分组信息
sa$ind <- factor(sa$ind)#转换为因子
sa$gr <- factor(sa$gr)
count <- as.matrix(count)#转换为矩阵DESeqDataSetFromMatrix()函数要求表达量数据为矩阵
ddsFullCount <- DESeqDataSetFromMatrix(countData = count,
                                       colData = sa,
                                       design = ~ gr)##生成表达量，分组信息，实验设计的数据集
dds <- DESeq(ddsFullCount)##进行差异分析
res <- results(dds, contrast = c("gr","Testis","Epididymis"))#指定对比，提取差异分析结果
res <- as.data.frame(res)#将差异分析结果转换为数据框
#每一步完成都看一下生成了什么
res$geneid <- rownames(res)
head(res)
```

    ##                       baseMean log2FoldChange     lfcSE       stat       pvalue
    ## ENSSSCG00000028996 12644.93781     -6.4681872 0.3130724 -20.660354 7.878488e-95
    ## ENSSSCG00000005267  1870.96399     -3.9051776 0.3708699 -10.529777 6.298374e-26
    ## ENSSSCG00000005268    27.24345      0.9896058 0.3294878   3.003468 2.669219e-03
    ## ENSSSCG00000005269    28.63889      1.4777168 0.4338131   3.406345 6.583904e-04
    ## ENSSSCG00000031382   550.77446      1.9870590 0.1643112  12.093264 1.146406e-33
    ## ENSSSCG00000005271  1040.12339     -4.0574608 0.3633861 -11.165701 6.001576e-29
    ##                            padj             geneid
    ## ENSSSCG00000028996 5.998575e-93 ENSSSCG00000028996
    ## ENSSSCG00000005267 4.811671e-25 ENSSSCG00000005267
    ## ENSSSCG00000005268 4.096284e-03 ENSSSCG00000005268
    ## ENSSSCG00000005269 1.082247e-03 ENSSSCG00000005269
    ## ENSSSCG00000031382 1.229232e-32 ENSSSCG00000031382
    ## ENSSSCG00000005271 5.269116e-28 ENSSSCG00000005271

## 3.2 画火山图

``` r
show.name <- data.frame(show=c("DEFB115","DEFB136","DEFB129","DEFB134","INSL3","CCDC36","TDRD1","PIWIL1","SPATA46","CATSPERD","SUN5",
                               "RPL10L","PIWIL2","DDX4","CCT6B","YBX2","TNP2","MAEL","MYCBPAP","FKBP6")) ##展示火山图上的基因名
geneinfo %>% 
  dplyr::select(geneid='Gene stable ID',genename='Gene name') %>% #选择需要的列，因为count矩阵没有基因名，所以要获取基因名
  distinct() %>% #去重复
  inner_join(.,show.name,by=c("genename"="show")) %>% #一步一步运行，看产生了什么
  full_join(.,res,by="geneid") %>% 
  dplyr::mutate(sig=ifelse(log2FoldChange>1 & padj<0.05,"up",
                           ifelse(log2FoldChange< -1 & padj<0.05,"dw","nosig"))) -> volcano.data#生成火山图数据
show.data <- volcano.data %>% filter(!is.na(genename))#生成要显示基因的数据
ggplot(volcano.data,aes(x=log2FoldChange,y= -1*log10(padj),label=genename)) + #哪一列做x轴，哪一列做y轴，y轴做了什么处理
  geom_point(aes(color=sig)) +#颜色映射
  geom_vline(xintercept = c(-1,1), linetype="dashed") +#x轴的虚线
  geom_vline(xintercept = c(-4,4), linetype="dashed") +
  geom_hline(yintercept = -1*log10(0.05), linetype="dashed") +#y轴的虚线
  geom_text_repel(data=show.data,aes(x=log2FoldChange,y= -1*log10(padj),label = genename),
                    box.padding = 0.5,min.segment.length = 0) +#添加基因标签
  geom_point(data = show.data,aes(x=log2FoldChange,y= -1*log10(padj)),shape=21,color="black") +#展示的基因点用圆圈框起
  scale_fill_manual(values = c(up="#f1abc9",dw="#5db2ff",nosig="#8d9eae"))+
  scale_color_manual(values = c(up="#f1abc9",dw="#5db2ff",nosig="#8d9eae"))+
  theme_test() +
  xlim(c(-20,20)) ##warning信息不用管
```

![](/images/posts/replotBOR/unnamed-chunk-9-1.png)<!-- -->

------------------------------------------------------------------------

# 4 Figure5

## 4.1 Fig5A

``` r
sqanti3_class %>% 
  inner_join(.,abun2,by=c("isoform"="pbid")) %>% 
  dplyr::select(stru=structural_category,GW.count,FG.count) %>% 
  pivot_longer(cols = 2:3,names_to = "sample") %>% 
  filter(value>0) %>% 
  mutate(type=ifelse(stru=="full-splice_match" | stru=="incomplete-splice_match","Annotated isoforms",
                     ifelse(stru=="novel_in_catalog"|stru=="novel_not_in_catalog","Novel isoforms from annotated genes",
                            ifelse(stru=="antisense","Novel isoforms from antisense genes",
                                   "Novel isoforms from novel genes")))) %>% 
  dplyr::group_by(sample) %>% 
  dplyr::count(type) %>% 
  dplyr::mutate(per=round(n/sum(n)*100,2)) -> fig5a.data
par(mfrow=c(1,2)) # 1行2列的画布
pie(fig5a.data[fig5a.data$sample=="GW.count",]$n,
    labels = paste(fig5a.data[fig5a.data$sample=="GW.count",]$type,"\n","(",fig5a.data[fig5a.data$sample=="GW.count",]$n,", ",fig5a.data[fig5a.data$sample=="GW.count",]$per,"%)"),
    col = c("purple", "violetred1", "green3","cyan"),
    main = "Testis")
pie(fig5a.data[fig5a.data$sample=="FG.count",]$n,
    labels = paste(fig5a.data[fig5a.data$sample=="FG.count",]$type,"\n","(",fig5a.data[fig5a.data$sample=="FG.count",]$n,", ",fig5a.data[fig5a.data$sample=="FG.count",]$per,"%)"),
    col = c("purple", "violetred1", "green3","cyan"),
    main = "Epididymis")
```

![](/images/posts/replotBOR/unnamed-chunk-10-1.png)<!-- -->

``` r
#画新的图时要关闭画布#dev.off()
```

## 4.2 Fig5C

读入四种方法预测的non-coding结果并处理

看看`inner_join()`和`full_join()`以及`merge()`的用法和区别

``` r
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
  dplyr::mutate(type=ifelse(X13<=0.001,"coding","noncoding")) %>%
  filter(type=="coding") %>% 
  dplyr::select(pbid="X1",type) %>% 
  distinct() %>% #去除重复
  full_join(.,abun2,by="pbid") %>% 
  filter(is.na(type))
```

绘图

``` r
##venn
#GW
venn.diagram(x = list(CNCI=cnci[cnci$GW.count>0,]$pbid,CPC2=cpc2[cpc2$GW.count>0,]$pbid,Pfam=pfam[pfam$GW.count>0,]$pbid,CPAT=cpat[cpat$GW.count>0,]$pbid),
             filename = NULL,
             fill=c("#57c1f1","#c09968","#a5cf4c","#a238e4"),
             alpha=0.6,fontfamily="sans",cat.fontfamily="sans",
             col="black",
             lwd=0.1,
             margin=0.1,
             height=3,
             width=3) -> nc.gw.venn
venn.diagram(x = list(CNCI=cnci[cnci$FG.count>0,]$pbid,CPC2=cpc2[cpc2$FG.count>0,]$pbid,Pfam=pfam[pfam$FG.count>0,]$pbid,CPAT=cpat[cpat$FG.count>0,]$pbid),
             filename = NULL,
             fill=c("#57c1f1","#c09968","#a5cf4c","#a238e4"),
             alpha=0.6,fontfamily="sans",cat.fontfamily="sans",
             col="black",
             lwd=0.1,
             margin=0.1,
             height=3,
             width=3) -> nc.fg.venn

cowplot::plot_grid(nc.gw.venn,nc.fg.venn,labels = c("Testis","Epididymis"))
```

![](/images/posts/replotBOR/unnamed-chunk-12-1.png)<!-- -->

------------------------------------------------------------------------

# 5 完了😋
