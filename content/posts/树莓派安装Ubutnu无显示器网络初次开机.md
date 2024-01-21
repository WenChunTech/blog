---
title: "树莓派安装Ubutnu无显示器网络初次开机"
subtitle: ""
date: 2022-10-24T23:29:49+08:00
draft: false
author: ""
authorLink: ""
authorEmail: ""
description: ""
keywords: ""
license: ""
comment: false
weight: 0

tags:
- raspberry
categories:
- 教程

hiddenFromHomePage: false
hiddenFromSearch: false

summary: ""
resources:
- name: featured-image
  src: featured-image.jpg
- name: featured-image-preview
  src: featured-image-preview.jpg

toc:
  enable: true
math:
  enable: false
lightgallery: false
seo:
  images: []

repost:
  enable: true
  url: ""

# See details front matter: https://fixit.lruihao.cn/theme-documentation-content/#front-matter
---

<!--more-->
# 安装镜像

下载地址： `https://ubuntu.com/download/raspberry-pi`

# 刷入镜像到SD卡

1. 刷入后需要重新拔插SD卡

2. 找到system-boot盘符盘符下network-config文件

3. 根据实际情况修改wifi配置
```yanl
version: 2
ethernets:
  eth0:
    dhcp4: true
    optional: true
#wifis:
#  wlan0:
#    dhcp4: true
#    optional: true
#    access-points:
#      myhomewifi:
#        password: "S3kr1t"
#      myworkwifi:
#        password: "correct battery horse staple"
#      workssid:
#        auth:
#          key-management: eap
#          method: peap
#          identity: "me@example.com"
#          password: "passw0rd"
#          ca-certificate: /etc/my_ca.pem
```

# 开机查找wifi

查找树莓派ip地址可用方法：

1. 登录路由器后台查看连接设备中树莓派IP

2. 使用Advanced_IP_Scanner软件

# 连接ssh
初始用户名和密码都是：`ubuntu`

```bash
ssh ubuntu@ip
```

> 注意：第一次登陆会强制要求修改密码，修改后再次登录即可，密码也不能太短

# 换源

1. 备份

```bash
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
```

2. 修改文件内容为:

```txt
deb http://mirrors.cloud.tencent.com/ubuntu-ports jammy main restricted
deb http://mirrors.cloud.tencent.com/ubuntu-ports jammy-updates main restricted
deb http://mirrors.cloud.tencent.com/ubuntu-ports jammy universe
deb http://mirrors.cloud.tencent.com/ubuntu-ports jammy-updates universe
deb http://mirrors.cloud.tencent.com/ubuntu-ports jammy multiverse
deb http://mirrors.cloud.tencent.com/ubuntu-ports jammy-updates multiverse
deb http://mirrors.cloud.tencent.com/ubuntu-ports jammy-backports main restricted universe multiverse
deb http://mirrors.cloud.tencent.com/ubuntu-ports jammy-security main restricted
deb http://mirrors.cloud.tencent.com/ubuntu-ports jammy-security universe
deb http://mirrors.cloud.tencent.com/ubuntu-ports jammy-security multiverse
```

3. 更新
```bash
sudo apt update
sudo apt upgrade -y
```

**参考链接：**

- [树莓派折腾记（一）-- 安装系统（Ubuntu Server 20.04.1） | 修行的技术阁 (mrxiuxing.com)](https://mrxiuxing.com/posts/2f81a42d.html)

