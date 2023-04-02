---
title: Stable_Diffusion_Model
subtitle:
date: 2023-03-30T23:16:41+08:00
draft: true
author:
  name:
  link:
  email:
  avatar:
description:
keywords:
license:
comment: false
weight: 0
tags:
  - draft
categories:
  - draft
hiddenFromHomePage: false
hiddenFromSearch: false
summary:
resources:
  - name: featured-image
    src: featured-image.jpg
  - name: featured-image-preview
    src: featured-image-preview.jpg
toc: true
math: false
lightgallery: false
password:
message:
repost:
  enable: true
  url:

# See details front matter: https://fixit.lruihao.cn/documentation/content/#front-matter
---

<!--more-->
# Stable Diffusion Model

## 一. 原理

1. diffusion整体思路如下:

![](https://pic.imgdb.cn/item/642706d1a682492fcc74e9bd.jpg)

说明: 整个过程主要分为正向过程和逆向过程.正向过程主要是将图像转化为纯噪声的过程,而逆向过程正好相反,是将纯噪声还原为原图像的过程.

- 正向过程:对于一张图像$\alpha_0$我们为它添加一个服从标准正态分布的噪声$z_0$,然后再在此基础上添加噪声$z_1$,每次添加的噪声都比上一次添加的噪声多,重复此操作,直到变为纯噪声$z_n$,此过程可以引出公式:

    $\alpha_t = 1 - \beta_t$ 

    $x_t = \sqrt{\alpha_t}x_{t-1} + \sqrt{1-\alpha_t} z_1$, 

    其中$\beta$需要越来越大,从而$\alpha$越来越小,可以将$\sqrt{1-\alpha_t}$理解为噪声的权重,这样每次生成的噪声都比上一次多

- 逆向过程:需要生成一个服从标准正态分布的噪声,然后再在此基础上进行去噪,得到上一步的图像,重复此操作得到最原始的图像$x_0$

2. 实现细节如下:

    - 正向过程:可以思考一下在$x_1$时刻的图像受到那些因素影响呢?答案很明显,受到前一刻图像和所加噪声的影响.从公式:

        $\alpha_t = 1 - \beta_t$ 
    
        $x_t = \sqrt{\alpha_t}x_{t-1} + \sqrt{1-\alpha_t} z_t$, 可以明显看出来.按照这个思路我们可以一步一步从$x_0 \rightarrow x_1 \rightarrow x_2 \rightarrow ... \rightarrow x_n$生成.那这样是不是计算量很大呢?有没有简单的方法直接从$x_0$生成到$x_n$呢?答案是可以的.下面详细推导一下.
        
        已知:
        
        ​    $x_t = \sqrt{\alpha_t}x_{t-1} + \sqrt{1-\alpha_{t-1}} z_t$
        
        ​    $x_{t-1} = \sqrt{\alpha_{t-1}}x_{t-2} + \sqrt{1-\alpha_{t-1}} z_{t-1}$
        
        可推导出:
        
        $$\begin{split}
        x_t &= \sqrt{\alpha_t}(\sqrt{\alpha_{t-1}}x_{t-2} + \sqrt{1-\alpha_{t-1}}z_{t-2}) + \sqrt{1-\alpha_t}z_{t-1} \\
        &= \sqrt{\alpha_t \alpha_{t-1}}x_{t-2} + \sqrt{\alpha_t (1-\alpha_{t-1})}z_{t-2}) + \sqrt{1-\alpha_t}z_{t-1} \\ 
        &= \sqrt{\alpha_t \alpha_{t-1}}x_{t-2} + \sqrt{1-\alpha_t \alpha_{t-1}} \hat {z})\\ \vdots \\ &= \sqrt{\bar \alpha_t}x_0 + \sqrt{1-\bar \alpha_t} \hat z \end{split}$$
        
        其中: $\bar \alpha_t$表示累乘即$\prod \limits_{i=0} \limits^t {\alpha_i}$, $\hat z$表示服从**标准正态分布**
        
        现在详细讲解关键推导过程:
        
        已知:
        
        ​    $\mathcal N( \mu, \sigma^2) = \mathcal N(0, 1) 标准正态分布$
        
        ​    $z_0,z_1, ..., z_n \sim \mathcal N(0, 1)$
        
        则:
        
        ​    $\sqrt{\alpha_t(1-\alpha_{t-1})}z_{t-2}) \sim \mathcal N(0, \alpha_t(1-\alpha_{t-1}) $
        
        ​    $\sqrt{1-\alpha_t}z_{t-1} \sim \mathcal N(0, 1-\alpha_t)$
        
        根据正态分布性质,上面2式相加可得:
        
        ​    $\sqrt{1-\alpha_t \alpha_{t-1}} \hat {z}) \sim \mathcal N(0, 1-\alpha_t \alpha_{t-1})$
        
        由此可知$\hat z $也是服从标准正态分布.
    
    

## 二. 示例



**参考链接:**

- [Diffusion Model原理详解及源码解析_秃头小苏的博客-CSDN博客](https://blog.csdn.net/qq_47233366/article/details/128508412)
