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

   - tun 设备: TUN设备是一种虚拟网络设备，用于在用户空间和内核空间之间传递网络数据。TUN代表“网络隧道”（Network TUNnel），它允许用户空间的程序通过标准的网络套接字接口发送和接收IP数据包，同时内核会处理网络协议的部分。TUN设备通常用于创建虚拟私有网络（VPN）和其他网络隧道，以在用户空间中运行的程序与内核之间传递网络流量。这对于实现网络隔离、虚拟专用网络、安全隧道等应用场景非常有用。一般来说，使用TUN设备的流程如下：

      1. 用户空间程序通过TUN设备发送IP数据包。
      2. 内核将接收到的数据包从TUN设备中读取。
      3. 用户空间程序可以通过标准的套接字接口监听和处理这些数据包。

   - tap 设备：TAP设备（Tap虚拟网卡）是一种虚拟网络设备，与TUN设备类似，也用于在用户空间和内核空间之间传递网络数据。TAP代表“网络透明适配器”（Network TAP），与TUN设备不同的是，TAP设备在数据链路层（OSI模型的第二层）工作，而TUN设备在网络层（第三层）工作。主要区别在于：

      1. TAP设备：它以太网设备的形式存在，可以处理链路层的数据包。因此，TAP设备可以传输以太网帧，包括处理MAC地址等。TAP设备通常用于需要在二层进行操作的场景，比如虚拟局域网（VLAN）的实现、桥接网络、以太网隧道等。

      2. TUN设备：与TAP不同，TUN设备是在网络层工作的，只能传输IP数据包。TUN设备通常用于实现虚拟私有网络（VPN）等应用，它只关心IP层的数据，而不涉及链路层的细节。

在使用TAP设备时，用户空间程序可以像处理真实的网络接口一样处理TAP设备，包括发送和接收以太网帧。

3. iptables 简单使用:
```bash
# 设置从eth0出去的数据包,动态将内部地址转为外部可访问地址
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
# 设置从tun0出去的数据包,动态将内部地址转为外部可访问地址
iptables -t nat -A POSTROUTING -o tun0 -j MASQUERADE
# 设置从eth0进来，状态为RELATED,ESTABLISHED 的数据包，允许进入
iptables -A INPUT -i eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
# 设置从tun0进来，状态为RELATED,ESTABLISHED 的数据包，允许进入
iptables -A INPUT -i tun0 -m state --state RELATED,ESTABLISHED -j ACCEPT
# 允许所以数据包转发
iptables -A FORWARD -j ACCEPT

```

4. iproute2 简单使用
```bash
# 设置tun0设备的ip为192.168.1.10/24
ip addr add 192.168.1.10/24 dev tun0

# 添加走到tun0设备的路由
ip route add 192.168

# 为table 100添加一条来自 192.168.1.10的规则
ip rule add from 192.168.1.10 table 100

```

## VPN 的简单实现
