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

1. perf top

   **常用参数**

   - -e：指定性能事件
   - -a：显示在所有 CPU 上的性能统计信息
   - -C：显示在指定 CPU 上的性能统计信息
   - -p：指定进程 PID
   - -t：指定线程 TID
   - -K：隐藏内核统计信息
   - -U：隐藏用户空间的统计信息
   - -s：指定待解析的符号信息
   - ‘‐G’ or‘‐‐call‐graph’ <output_type,min_percent,call_order>
   - graph: 使用调用树，将每条调用路径进一步折叠。这种显示方式更加直观。
   - 每条调用路径的采样率为绝对值。也就是该条路径占整个采样域的比率。

2. perf list

3. perf test

4. perf record: 记录一段时间内系统/进程的性能时间

   **常用参数**

   - -e：选择性能事件

   - -p：待分析进程的 id

   - -t：待分析线程的 id

   - -a：分析整个系统的性能

   - -C：只采集指定 CPU 数据

   - -c：事件的采样周期

   - -o：指定输出文件，默认为 perf.data

   - -A：以 append 的方式写输出文件

   - -f：以 OverWrite 的方式写输出文件

   - -g：记录函数间的调用关系

5. perf report: 读取 perf record 生成的数据文件，并显示分析数据

   **常用参数**

   - -i：输入的数据文件
   - -v：显示每个符号的地址
   - -d <dos>：只显示指定 dos 的符号
   - -C：只显示指定 comm 的信息（Comm. 触发事件的进程名）
   - -S：只考虑指定符号
   - -U：只显示已解析的符号
   - -g[type,min,order]：显示调用关系，具体等同于 perf top 命令中的-g
   - -c：只显示指定 cpu 采样信息
   - -M：以指定汇编指令风格显示
   - –source：以汇编和 source 的形式进行显示
   - -p<regex>：用指定正则表达式过滤调用函数

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
