---
title: Golang内存模型
subtitle:
date: 2024-04-18T20:07:25+08:00
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
  - 内存模型
categories:
  - Golang
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

# Golang内存模型

## 介绍

Golang内存模型是Golang程序员必须了解的一个重要概念，它定义了Golang程序中的内存访问规则，保证了Golang程序的正确性。Golang内存模型是基于happens-before关系的，happens-before关系是一个偏序关系，它定义了程序中事件的顺序关系。在Golang内存模型中，happens-before关系定义了内存访问的顺序关系，保证了Golang程序的正确性。

## happens-before关系

happens-before是很多语言都拥有的一个术语，定义通常是：假设A和B表示一个多线程的程序执行的两个操作。如果A happens-before B，那么A操作对内存的影响 将对执行B的线程(且执行B之前)可见。

它有几个特点：
1. A happens-before B并不意味着A在B之前发生
2. A在B之前发生并不意味着A happens-before B
3. 具有传递性：如果A happens-before B，B happens-before C，那么A happens-before C

> 注意：happens-before并不是指时序关系，并不是说A happens-before B就表示操作A在操作B之前发生，它就是一个术语，就像光年不是时间单位一样

## happens-before关系在Golang中的场景定义

1. 如果操作A和B在相同的线程中执行，并且A操作的声明在B之前，那么A happens-before B。（这条规则基本上大部分语言都适用）


## 参考链接

1. https://tiancaiamao.gitbooks.io/go-internals/content/zh/10.1.html
2. https://go.cyub.vip/concurrency/memory-model/
3. https://research.swtch.com/plmm
