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

- 逆向过程:

$$x_{t-1} = \sqrt{\alpha_{t-1}}x_{t-2} + \sqrt{1-\alpha_{t-1}} z_2$$

$$\begin{split}
x_t &= \sqrt{\alpha_t}(\sqrt{\alpha_{t-1}}x_{t-2} + \sqrt{1-\alpha_{t-1}}z_2) + \sqrt{1-\alpha_t}z_1 \\
&= \sqrt{\alpha_t \alpha_{t-1}}x_{t-2} + \sqrt{\alpha_t (1-\alpha_{t-1})}z_2) + \sqrt{1-\alpha_t}z_1 \\ 
&= \sqrt{\alpha_t \alpha_{t-1}}x_{t-2} + \sqrt{\alpha_t (1-\alpha_t \alpha_{t-1})} \bar {z_2}) \end{split}$$

## 二. 示例



**参考链接:**

- [Diffusion Model原理详解及源码解析_秃头小苏的博客-CSDN博客](https://blog.csdn.net/qq_47233366/article/details/128508412)



Diffusion model是一种生成模型，它可以从简单的高斯噪声生成复杂的数据样本，如图像、文本、语音等。Diffusion model的基本原理是通过一个前向扩散过程和一个后向去噪过程来实现数据的生成。前向扩散过程是将真实数据逐步加入高斯噪声，直到变成纯噪声；后向去噪过程是利用一个神经网络来逐步消除噪声，直到恢复出真实数据。在这篇博客中，我们将详细讲解diffusion model的原理、优势、应用和代码实现。

前向扩散过程

前向扩散过程的目的是将真实数据分布q(x)转换为一个简单的高斯分布p(x_T)，其中x_T是最终的纯噪声。这个过程可以分为T个步骤，每个步骤都会给数据加入一定比例的高斯噪声，使得数据分布逐渐变得平滑和简单。具体来说，给定一个真实数据样本x_0，我们可以按照以下公式进行前向扩散：

$x_t = \sqrt{1 - beta_t} * x_(t-1) + \sqrt{beta_t} * epsilon_t$

其中epsilon_t ~ N(0, I)是一个标准正态分布的随机变量，beta_t是一个预先定义好的噪声比例参数，满足0 < beta_t < 1，并且随着t的增加而增加。这样，当t=0时，x_0就是原始数据；当t=T时，x_T就是纯噪声。我们可以用下图来直观地展示前向扩散过程：

![image](https://pic4.zhimg.com/80/v2-9b8c6f5a2f7d6c3a9f8a7b5c6b3e9d4a_1440w.jpg)

后向去噪过程

后向去噪过程的目的是将纯噪声p(x_T)转换为真实数据分布q(x_0)，这个过程也可以分为T个步骤，每个步骤都会利用一个神经网络来预测并消除一定比例的噪声，使得数据分布逐渐变得复杂和真实。具体来说，给定一个纯噪声样本x_T，我们可以按照以下公式进行后向去噪：

x_(t-1) = mu_theta(x_t, t) + sigma_t * epsilon_(t-1)

其中epsilon_(t-1) ~ N(0, I)是一个标准正态分布的随机变量，sigma_t是一个固定的标准差参数，mu_theta(x_t, t)是一个神经网络输出的均值函数，它可以根据当前的数据样本x_t和当前的步数t来预测下一步的数据样本x_(t-1)。这样，当t=T时，x_T就是纯噪声；当t=0时，x_0就是生成的数据。我们可以用下图来直观地展示后向去噪过程：

![image](https://pic2.zhimg.com/80/v2-5e6f4c6f8b7e5c8d9e3c5d9a7f4b3a6e_1440w.jpg)
