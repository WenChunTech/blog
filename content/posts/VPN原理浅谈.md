---
title: VPN原理浅谈
subtitle:
date: 2023-11-05T23:01:37+08:00
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
  - vpn
  - tun
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

# VPN 原理浅谈

## VPN 的作用

1. VPN 的作用是将用户的数据流量通过加密的方式传输到 VPN 服务器，然后再由 VPN 服务器转发到目标服务器，这样就可以达到隐藏用户真实 IP 的目的。

2. VPN 在企业中的作用是将企业内部的数据流量通过加密的方式传输到 VPN 服务器，然后再由 VPN 服务器转发到目标服务器，这样就可以达到保护企业内部数据的目的。
