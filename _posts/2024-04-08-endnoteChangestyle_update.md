---
layout: post
title: Endnote修改参考文献样式（更新）
description:
author: Z.P.L.
categories: Endnote
keywords: Software skill
---

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=1958106048&auto=1&height=66"></iframe>

此处使用的是**endnote 20**

endnote X9和20的界面不一样，但修改style的方法是一样的

# 样式选择

Tools -> Output Styles -> Open style manger... -> 选择所投期刊

如果没有找到，点击右下脚**Get More on the Web**

进入网站后搜索期刊
![screenshot](/images/posts/endnote/searchstyle.png)

如果**搜索不到**想投期刊的参考文献样式，此时需要自行修改创建

比如我们感兴趣**animal reproduction**，但在**endnote style**网站里搜索不到，我们可以创建一个全新的风格，但这种方法要了解endnote中的一些插入字段，不太方便，即便了解了这些字段，一段时间后也会忘记。一个有效的方法便是找一个与所投期刊相近的期刊格式进行修改，怎么样找相近的呢（实际上，选择一个模板（nature系）即可）

## endnote 20修改/创建样式方法
Tools --> Output Styles --> Open style manger... --> 选中一个期刊右键出现**edit style**,改成自己想要的期刊格式即可。

如**animal reproduction**的参考文献格式如下：

- Gastal EL, Gastal MO, Ginther OJ. Experimental assumption of dominance by a smaller follicle and associated hormonal changes in mares. Biol Reprod. 1999a;61(3):724-30. http://dx.doi.org/10.1095/biolreprod61.3.724. PMid:10456850.
- Gastal EL, Donadeu FX, Gastal MO, Ginther OJ. Echotextural changes in the follicular wall during follicle deviation in mares. Theriogenology. 1999b;52(5):803-14. http://dx.doi.org/10.1016/S0093-691X(99)00173-9. PMid:10735121.
- Hess RA, Carnes K. The role of estrogen in testis and the male reproductive tract: a review and species comparison. Anim Reprod. 2004;1:5-30.

改成如上的样式

选择与以上样式相似的期刊风格进行修改（可以随便选择一个）
![screensot](/images/posts/endnote/chosestyle.png)

Author. Title|. Journal| Volume|, Pages| (Year).| `https://doi.org`:DOI
![screensot](/images/posts/endnote/editstyle.png)



先看看**nature**风格插入word的效果
![screensot](/images/posts/endnote/natureRefStyle.png)

现在在endnote里修改为**animal reproduction**样式

可以看出需要修改作者, year加到前面，添加issuse, 页码缩写，加上PMID

如下所示：

Author. Title|. Journal. Year;Volume(Issue)|, Pages. `https://dx.doi.org`:DOI. PMid:Accession Number
![screensot](/images/posts/endnote/changedstyle.png)

修改完后，在word中测试看看，可以看到在word里已经改好了，但作者信息不很对

![screensot](/images/posts/endnote/styleinword.png)

还需要在endnote里改一下

![screensot](/images/posts/endnote/changeAuthor.png)

再改改page numbers就改完了

![screensot](/images/posts/endnote/finalStyleinWord.png)


## 最好的方法就是选择一个风格，在已有的风格上进行修改会容易很多

# 上标修改

在citation那里修改

![screensot](/images/posts/endnote/citeup.png)
