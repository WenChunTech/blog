---
title: "强大的GAN网络"
subtitle: ""
date: 2023-02-25T22:54:43+08:00
draft: false
author: ""
authorLink: ""
authorEmail: ""
description: ""
keywords: ""
license: ""
comment: false
weight: 0

tags:
- deeplearning
- gan
categories:
- Python

hiddenFromHomePage: false
hiddenFromSearch: false

summary: ""
resources:
- name: featured-image
  src: featured-image.jpg
- name: featured-image-preview
  src: featured-image-preview.jpg

toc:
  enable: true
math:
  enable: false
lightgallery: false
seo:
  images: []

repost:
  enable: true
  url: ""

# See details front matter: https://fixit.lruihao.cn/theme-documentation-content/#front-matter
---

<!--more-->

# 强大的GAN网络

## 1. 概要

​	GAN网络全称**generative adversarial network**,翻译为生成式对抗网络,是一种非监督式学习机器学习方法。由`Ian J,Goodfello` 等人于2014年在[Generative Adversarial Nets](https://arxiv.org/pdf/1406.2661.pdf) 论文中提出。其中在GAN网络中,有两个模型——生成模型(generative model G),判别模型(discriminative model D).

## 2. 原理

​	GAN网络主要运用了博弈论的思想,模型中的2为博弈方分别由生成模型和判别模型担当.生成模型用随机取样作为输入,它的输出结果要尽可能和训练样本尽可能相似,最好的情况就是分辨不出是真实样本还是生成出来的样本.而判别模型就是尽可能判别生成模型生成的结果和真实样本.这样2个网络相互对抗,不断调整参数,最终达到纳什均衡.

这个过程可以表示为:
$$
min_G max_DV(D,G) = \Epsilon_{x\sim P_{data}(x)}[logD(x)] + \Epsilon_{z\sim p_{z}(z)}[log(1-D(G(z)))]
$$
公式解释:

	1. 当训练D时,希望这个式子的值越大越好.真实数据希望被D分成1,生成数据希望被分成0
	1. 当训练G时,希望这个式子的值越小越好.希望D分不开真实数据还是生成数据

>  零和博弈（zero-sum game），又称零和游戏，与非零和博弈相对，是博弈论的一个概念，属非合作博弈。指参与博弈的各方，在严格竞争下，一方的收益必然意味着另一方的损失，博弈各方的收益和损失相加总和永远为“零”，双方不存在合作的可能。就像下棋的游戏一样，你走的每一步和对方走的每一步都是向着对自己有利的方向走，然后你和对手轮流走步
> 每一步都向着自己最大可能能赢的地方走。这就是零和博弈。



## 3. 简单代码实现

```python
```



参考资料:

1. [GAN入门理解及公式推导 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/28853704)
