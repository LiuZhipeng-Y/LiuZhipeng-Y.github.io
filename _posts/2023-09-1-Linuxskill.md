---
layout: post
title: Linux 注意
description: Linux中grep的使用技巧。
author: Z.P.L.
categories: Linux
keywords: Linux code
topmost: false
---


## Linux中方括号

使用[[ ... ]]条件判断结构，而不是[ ... ]，能够防止脚本中的许多逻辑错误。比如，&&、||、<和> 操作符能够正常存在于[[ ]]条件判断结构中，但是如果出现在[ ]结构中的话，会报错。

如：可以直接使用if [[ $a != 1 && $a != 2 ]], 如果不使用双括号，则为if [ $a -ne 1 ] && [ $a != 2 ]或者if [ $a -ne 1 -a $a != 2 ]
