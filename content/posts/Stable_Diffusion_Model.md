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

$\alpha_t = 1 - \beta_t$, 其中$\beta$需要越来越大,从而$\alpha$越来越小

$$x_t = \sqrt{\alpha_t}x_{t-1} + \sqrt{1-\alpha_t} z_1$$

$$x_{t-1} = \sqrt{\alpha_{t-1}}x_{t-2} + \sqrt{1-\alpha_{t-1}} z_2$$

$$\begin{split}
x_t &= \sqrt{\alpha_t}(\sqrt{\alpha_{t-1}}x_{t-2} + \sqrt{1-\alpha_{t-1}}z_2) + \sqrt{1-\alpha_t}z_1 \\
&= \sqrt{\alpha_t \alpha_{t-1}}x_{t-2} + \sqrt{\alpha_t (1-\alpha_{t-1})}z_2) + \sqrt{1-\alpha_t}z_1 \\ 
&= \sqrt{\alpha_t \alpha_{t-1}}x_{t-2} + \sqrt{\alpha_t (1-\alpha_t \alpha_{t-1})} \bar {z_2}) 

\end{split}$$


## 二. 示例