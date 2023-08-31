---
title: Docker基石 NetworkNamespace
subtitle:
date: 2023-08-31T21:40:12+08:00
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
  - veth
  - docker
categories:
  - linux
  - docker
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

# Docker 基石-NetworkNamespace

## 1. veth-pair 是什么

veth-pair 是一对虚拟网卡，通过 veth-pair 可以将两个网络命名空间连接起来，从而实现两个网络命名空间之间的通信。和 tap/tun 设备不同的是，它都是成对出现的。一端连着协议栈，一端彼此相连着

> 全程只使用 ip 命令进行演示

![](https://pic.imgdb.cn/item/64f09a70661c6c8e5415497a.jpg)
![](https://pic.imgdb.cn/item/64f09ac3661c6c8e54155968.jpg)

## 2. 使用 veth-pair 和 netns 进行演示

### 2.1 创建两个网络命名空间

```bash
ip netns add ns1
ip netns add ns2
```

### 2.2 创建 veth-pair

```bash
ip link add veth1 type veth peer name veth2
```

### 2.3 将 veth1 移动到 ns1 中

```bash
ip link set veth1 netns ns1
```

### 2.4 将 veth2 移动到 ns2 中

```bash
ip link set veth2 netns ns2
```

### 2.5 启动 veth1

```bash
ip netns exec ns1 ip link set dev veth1 up
```

### 2.6 启动 veth2

```bash
ip netns exec ns2 ip link set dev veth2 up
```

### 2.7 设置 veth1 的 IP 地址

```bash
ip netns exec ns1 ip addr add 192.168.1.2/24 dev veth1
```

### 2.8 设置 veth2 的 IP 地址

```bash
ip netns exec ns2 ip addr add 192.168.1.3/24 dev veth2
```

### 2.9 测试

```bash
ip netns exec ns1 ping 192.168.1.3
```

![](https://pic.imgdb.cn/item/64f09f57661c6c8e5416cc98.jpg)
