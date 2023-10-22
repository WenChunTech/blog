---
title: Perf性能Profiling
subtitle:
date: 2023-10-22T14:10:29+08:00
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
  - perf
  - linux
categories:
  - linux
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

# 性能 Profiling -- 使用 Perf 命令

## 1. Perf 常用命令

- perf top
- perf list
- perf test
- perf record
- perf report

## 2. 生成火焰图示例

```bash

# 采集数据
perf record -F 99 -p 1234 -g <command> -- sleep 30

# 解析数据
perf script -i perf.data &> perf.unfold

# 生成火焰图

# 克隆仓库
git clone https://github.com/brendangregg/FlameGraph.git
cd FlameGraph

# 对perf.unfold文件进行符号折叠
./stackcollapse-perf.pl perf.unfold > perf.folded

# 生成火焰图（svg图）
./flamegraph.pl perf.folded > perf.svg

```
