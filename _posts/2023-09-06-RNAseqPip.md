---
layout: post
title: RNAseq流程--基于购买的服务器
description: pipeline for RNA-seq
author: Z.P.L.
categories: pipeline
keywords:  RNA-seq R bioinfo Linux
---

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=520 height=52 src="//music.163.com/outchain/player?type=2&id=2001434&auto=1&height=32"></iframe>

## fastp质控

``` sh
source activate rna
fastp -i ST2-4A_1.fq.gz -I ST2-4A_2.fq.gz -o ST2-4A_1.good.fq.gz -O ST2-4A_2.good.fq.gz -h ST2.html
```

``` sh
#salmon index
salmon index -t genome.fa -d chromesome.name.txt -i index_name -p 20
```

``` sh
salmon quant \
	--gcBias \
	-l A \
	-1 reads1.fq.gz \
	-2 reads2.fq.gz \
	-i index_name \
	-g gtf \
	-o output \
	-p 20 \
```



see more detail <https://github.com/sunyumail93/PipeRNAseq/blob/master/PipeRNAseq.sh>


