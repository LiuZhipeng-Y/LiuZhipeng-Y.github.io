---
layout: page
title: About
description: 打码改变世界
keywords: Zhipeng Liu, 刘志朋
comments: true
menu: 关于
permalink: /about/
---

## My articles

1. **Zhipeng Liu**, Hongmei Dai, Hailong Huo, Weizhen Li, Yun Jiang, Xia Zhang, Jinlong Huo. Molecular characteristics and transcriptional regulatory of spermatogenesis-related gene RFX2 in adult Banna mini-pig inbred line (BMI) [J]. Animal Reproduction, 2023, 20(1): e20220090. https://doi.org/10.1590/1984-3143-AR2022-0090

2. **刘志朋**，代红梅，杨忠，张霞，霍海龙，王配，李卫真，赵筱，霍金龙*.版纳微型猪近交系精子发生相关基因PHF7的分子特征及转录调控研究[J]．四川农业大学学报，2022, 40(2): 243-252．        
        
3. **刘志朋**，张霞，代红梅，杨福华，廖隽锐，许静，霍金龙*.基于转录组测序分析SPACA1在版纳微型猪近交系睾丸中的表达及转录调控特征[J]．河北农业大学学报，2023, 46(1): 88-96.        
        
        

坚信熟能生巧，努力改变人生。

## 联系我

<ul>
{% for website in site.data.social %}
<li>{{website.sitename }}：<a href="{{ website.url }}" target="_blank">@{{ website.name }}</a></li>
{% endfor %}
{% if site.url contains 'mazhuang.org' %}
<li>
微信公众号：<br />
<img style="height:192px;width:192px;border:1px solid lightgrey;" src="{{ site.url }}/assets/images/qrcode.jpg" alt="搬砖" />
</li>
{% endif %}
</ul>


## Skill Keywords

{% for skill in site.data.skills %}
### {{ skill.name }}
<div class="btn-inline">
{% for keyword in skill.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
