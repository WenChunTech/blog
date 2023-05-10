---
title: SelfAttention和MultiHeadAttention流程简述
subtitle:
date: 2023-05-07T11:38:09+08:00
draft: false
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

# SelfAttention和MultiHeadAttention流程简述

## 1. SelfAttention

- **1.1** 对于每一个输入$I$,初始化3个权重矩阵$W_q,W_k,W_v$.
- **1.2** 输入$I$分别与3个权重矩阵相乘得到$Q,K,V$
- **1.3** 将$Q$与$K$相乘(矩阵乘法)得到$A$(attention score)
- **1.4** $A$经过softmax激活得到$A'$
- **1.5** 将$A'$与$V$相乘得到B

**代码示例**:

 ```ipython
 In [16]: i = torch.randn(3,4)
 
 In [17]: w_query = torch.randn(4, 2)
 
 In [18]: w_key = torch.randn(4, 2)
 
 In [19]: w_value = torch.randn(4, 2)
 
 In [20]: i
 Out[20]:
 tensor([[ 0.4313,  1.3749, -0.2489,  1.3275],
         [-0.6467,  1.6335,  2.8923,  0.9124],
         [ 0.2326, -0.2314,  0.3554,  0.0892]])
 
 In [21]: w_query
 Out[21]:
 tensor([[ 0.3673, -0.8505],
         [-0.3559, -0.3708],
         [-1.2093,  0.6634],
         [ 0.4042,  0.8015]])
 
 In [22]: w_key
 Out[22]:
 tensor([[-1.1617,  0.4023],
         [-0.1249, -0.1605],
         [-0.8427,  1.1002],
         [-1.1320,  0.4611]])
 
 In [23]: w_value
 Out[23]:
 tensor([[-0.7675, -0.3179],
         [-1.1910, -0.6937],
         [-1.4809, -1.5502],
         [-0.7373,  1.0511]])
 
 In [24]: querys = i @ w_query
 
 In [25]: keys = i @ w_key
 
 In [26]: values = i @ w_value
 
 In [27]: querys
 Out[27]:
 tensor([[ 0.5066,  0.0223],
         [-3.9478,  2.5946],
         [-0.2259,  0.1952]])
 
 In [28]: keys
 Out[28]:
 tensor([[-1.9656,  0.2911],
         [-2.9229,  3.0806],
         [-0.6418,  0.5629]])
 
 In [29]: values
 Out[29]:
 tensor([[-2.5787,  0.6903],
         [-6.4049, -4.4522],
         [-0.4950, -0.3706]])
 
 In [30]: att_scores = querys @ keys.T
 
 In [31]: att_scores
 Out[31]:
 tensor([[-0.9893, -1.4121, -0.3126],
         [ 8.5152, 19.5319,  3.9940],
         [ 0.5009,  1.2617,  0.2549]])
 
 In [32]: att_scores_softmax = torch.nn.functional.softmax(att_scores, dim=-1)
 
 In [33]: att_scores_softmax
 Out[33]:
 tensor([[2.7604e-01, 1.8087e-01, 5.4310e-01],
         [1.6426e-05, 9.9998e-01, 1.7865e-07],
         [2.5498e-01, 5.4565e-01, 1.9937e-01]])
 
 In [34]: outputs = att_scores_softmax @ values
 
 In [35]: outputs
 Out[35]:
 tensor([[-2.1390, -0.8160],
         [-6.4048, -4.4521],
         [-4.2510, -2.3272]])
 
 In [36]: outputs.shape
 Out[36]: torch.Size([3, 2])
 ```
