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

## VPN 的应用场景

1. 保护隐私： VPN 通过加密通信，确保你的互联网连接是安全的。这对于在公共 Wi-Fi 网络上浏览互联网时尤其重要，因为公共网络通常更容易受到黑客攻击。

2. 绕过地理限制： 有些网站或服务在特定地区可能不可用，或者内容受到地理限制。使用 VPN 可以改变你的 IP 地址，使你能够绕过这些地理限制，访问受限制的内容。

3. 保护公司内部网络： 公司可以使用 VPN 来建立安全的远程访问通道，员工可以通过 VPN 连接到公司内部网络，以便在远程地点安全地访问公司资源。

4. 匿名上网： VPN 可以隐藏你的真实 IP 地址，提高上网的匿名性。这对于那些希望在互联网上保持相对匿名的用户来说很有用。

5. 绕过审查： 在一些国家，互联网审查可能会限制对特定网站或服务的访问。使用 VPN 可以绕过这些审查，让用户能够自由地访问互联网。

6. 安全远程办公： 对于远程工作者或分布式团队来说，VPN 提供了一个安全的通信渠道，使他们能够远程连接到公司网络并访问必要的资源。

7. 防止数据被窃取： 在使用不安全的网络时，VPN 可以加密你的数据流，防止敏感信息被窃取，确保通信的机密性。

8. 游戏加速： 一些玩家可能使用 VPN 来连接到特定地区的游戏服务器，以获得更好的游戏性能或访问特定地区的游戏内容。

## VPN 的工作原理

1. 涉及到的底层技术

   - tun/tap: 用于创建虚拟网卡，用于接收和发送数据包, 主要的操作就是在此实现的
   - iptables： 用于实现数据包的转发
   - iproute2： 用于实现路由表的管理

2. tun/tap 概述：

   - tun 设备：

   - tap 设备：

3. iptables 简单使用:

4. iproute2 简单使用

## VPN 的简单实现
