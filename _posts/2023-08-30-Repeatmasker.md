---
layout: post
title: Repeatmasker重复元件分析
description:
author: Z.P.L.
categories: Linux software
keywords: biology
---

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=520 height=52 src="//music.163.com/outchain/player?type=2&id=17177306&auto=1&height=32"></iframe>

This is my software dir

``` sh
source activate newrep
/home/data/vip6t05/soft/RepeatMasker/RepeatMasker -h
```

Test for RepeatMasker

``` sh
RepeatMasker -parallel 10 -species primates -html -gff -dir Gibbon_NLE ~/zhoubin/data/diff_spe_assembly/NLE.fa
```

It's from https://github.com/kizzb/Great-apes-specific-SVs/blob/main/Total_script.md
