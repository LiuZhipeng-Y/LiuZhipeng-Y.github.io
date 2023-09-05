---
layout: post
title: Picard使用
description:
author: Z.P.L.
categories: Linux_software
keywords: Software skill linux bioinfomatics
---

picard有很多有用的工具，了解它的用法

## 调用Picard

``` sh
java -jar ~/soft/picard/build/libs/picard.jar -h
```

``` sh
#This command could be useful.
#same to bedtools getfasta
java -jar ~/soft/picard/build/libs/picard.jar ExtractSequences -h
```

## 去除PCR重复

``` sh
java -jar ~/soft/picard/build/libs/picard.jar MarkDuplicates -h
```
