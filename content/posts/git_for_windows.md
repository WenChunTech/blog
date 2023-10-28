---
title: "Git for Windows 安装 Pacman"
subtitle: ""
date: 2022-12-24T18:30:33+08:00
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
- git
- pacman
- msys2
categories:
- Git
- Windows

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

# Git for Windows 安装 Pacman

## **1. 下载必要压缩包**

```bash
curl -o pacman-6.0.1-9-x86_64.pkg.tar.zst -L https://repo.msys2.org/msys/x86_64/pacman-6.0.1-9-x86_64.pkg.tar.zst
curl -o pacman-mirrors-20221016-1-any.pkg.tar.zst -L https://repo.msys2.org/msys/x86_64/pacman-mirrors-20221016-1-any.pkg.tar.zst
curl -o msys2-keyring-1~20221024-1-any.pkg.tar.zst -L https://repo.msys2.org/msys/x86_64/msys2-keyring-1~20221024-1-any.pkg.tar.zst

# 正常来说只需要上面三个包但是由于缺少 zstd 解压工具还需要 zstd 包
curl -o zstd-1.5.2-2-x86_64.pkg.tar.zst -L https://repo.msys2.org/msys/x86_64/zstd-1.5.2-2-x86_64.pkg.tar.zst
# 但是又因为 zstd 包也是 zstd 打包又需要另一个不是 zstd 打包的解压工具来解压
curl -o zstd-v1.5.2-win64.zip -L https://github.com/facebook/zstd/releases/download/v1.5.2/zstd-v1.5.2-win64.zip
```

## **2 .解压并安装:**

```bash
unzip zstd-v1.5.2-win64.zip "zstd-v1.5.2-win64/zstd.exe" -d .
./zstd-v1.5.2-win64/zstd.exe -d -o zstd-1.5.2-2-x86_64.pkg.tar zstd-1.5.2-2-x86_64.pkg.tar.zst
tar -xvf zstd-1.5.2-2-x86_64.pkg.tar -C /
tar -xvf msys2-keyring-1~20221024-1-any.pkg.tar.zst -C /
tar -xvf pacman-mirrors-20221016-1-any.pkg.tar.zst -C /
tar -xvf pacman-6.0.1-9-x86_64.pkg.tar.zst -C /
```

## **3. 添加密钥并更新数据库:**

```bash
pacman-key --init && pacman-key --populate msys2

pacman-db-upgrade -r /../

pacman -Sy
```

## **4. 更新元数据:**

```bash
# pacman -S pacman-mirrors-20221016-1 msys2-keyring-1~20221024-1 zstd-1.5.2-2
# pacman -S $(cut -d ' ' -f 1 /etc/package-versions.txt)

URL=https://github.com/git-for-windows/git-sdk-64/raw/main
cat /etc/package-versions.txt | while read p v; do
  d=/var/lib/pacman/local/$p-$v
  mkdir -p $d
  for f in desc files install mtree; do
    echo "$URL$d/$f" ..... $d/$f
    curl -sSL "$URL$d/$f" -o $d/$f
  done
done

curl -sSL https://github.com/git-for-windows/git-sdk-64/raw/main/pacman-6.0.1-9/desc -o /var/lib/pacman/local/pacman-6.0.1-9/desc
curl -sSL https://github.com/git-for-windows/git-sdk-64/raw/main/pacman-6.0.1-9/files -o /var/lib/pacman/local/pacman-6.0.1-9/files
curl -sSL https://github.com/git-for-windows/git-sdk-64/raw/main/pacman-6.0.1-9/install -o /var/lib/pacman/local/pacman-6.0.1-9/install
curl -sSL https://github.com/git-for-windows/git-sdk-64/raw/main/pacman-6.0.1-9/mtree -o /var/lib/pacman/local/pacman-6.0.1-9/mtree

curl -sSL https://github.com/git-for-windows/git-sdk-64/raw/main/pacman-mirrors-20221016-1/desc -o /var/lib/pacman/local/pacman-mirrors-20221016-1/desc
curl -sSL https://github.com/git-for-windows/git-sdk-64/raw/main/pacman-mirrors-20221016-1/files -o /var/lib/pacman/local/pacman-mirrors-20221016-1/files
curl -sSL https://github.com/git-for-windows/git-sdk-64/raw/main/pacman-mirrors-20221016-1/install -o /var/lib/pacman/local/pacman-mirrors-20221016-1/install
curl -sSL https://github.com/git-for-windows/git-sdk-64/raw/main/pacman-mirrors-20221016-1/mtree -o /var/lib/pacman/local/pacman-mirrors-20221016-1/mtree

curl -sSL https://github.com/git-for-windows/git-sdk-64/raw/main/msys2-keyring-1~20221024-1/desc -o /var/lib/pacman/local/msys2-keyring-1~20221024-1/desc
curl -sSL https://github.com/git-for-windows/git-sdk-64/raw/main/msys2-keyring-1~20221024-1/files -o /var/lib/pacman/local/msys2-keyring-1~20221024-1/files
curl -sSL https://github.com/git-for-windows/git-sdk-64/raw/main/msys2-keyring-1~20221024-1/install -o /var/lib/pacman/local/msys2-keyring-1~20221024-1/install
curl -sSL https://github.com/git-for-windows/git-sdk-64/raw/main/msys2-keyring-1~20221024-1/mtree -o /var/lib/pacman/local/msys2-keyring-1~20221024-1/mtree

curl -sSL https://github.com/git-for-windows/git-sdk-64/raw/main/zstd-1.5.2-2/desc -o /var/lib/pacman/local/zstd-1.5.2-2/desc
curl -sSL https://github.com/git-for-windows/git-sdk-64/raw/main/zstd-1.5.2-2/files -o /var/lib/pacman/local/zstd-1.5.2-2/files
curl -sSL https://github.com/git-for-windows/git-sdk-64/raw/main/zstd-1.5.2-2/install -o /var/lib/pacman/local/zstd-1.5.2-2/install
curl -sSL https://github.com/git-for-windows/git-sdk-64/raw/main/zstd-1.5.2-2/mtree -o /var/lib/pacman/local/zstd-1.5.2-2/mtree
```

## **5.  安装Fish Shell**

### 5.1 安装必要依赖

```bash
pacman -S gcc-libs
```

### 5.2 安装Fish Shell并启动

```bash
pacman -S fish
```

### 5.3 启动Fish Shell

```bash
# 在~/.bashrc文件中添加以下内容，设置默认shell为fish
if [ -t 1 ]; then
exec fish
fi
```

### 5.4 更换Fish Shell 主题

```bash
fish_config
```

### 5. 5 Fish Shell 常用命令和基本设置

```bash
# 关闭问候语
set -g -x fish_greeting ''
set -U fish_greeting

# 设置环境变量
#在最后一行加入(注意目录间用空格隔开)
set -x PATH /opt/demo/bin /home/guest/bin $PATH

# 删除变量
set -e Foo

# 配置别名
# Define alias in config file
alias rmi="rm -i"

# 切换默认shell，git for windows下不可用
echo $SHELL
cat /etc/shells
chsh -s /usr/bin/fish

# 命令替换
echo (date)
# 在bash下面
# echo `date`


```



**参考资料:**

- [pacman常用命令-昨夜星辰 (hustlei.github.io)](https://hustlei.github.io/2018/11/msys2-pacman.html)
- [Install inside MSYS2 proper · git-for-windows/git Wiki (github.com)](https://github.com/git-for-windows/git/wiki/Install-inside-MSYS2-proper)
- [Git:給 git for windows 裝個翅膀 (安裝 pacman 及其他工具) @ 傑克! 真是太神奇了! :: 痞客邦 :: (pixnet.net)](https://magicjackting.pixnet.net/blog/post/222933984)
- [在 Windows 的 Git Bash 中使用包管理器 - iris (ginshio.org)](https://blog.ginshio.org/2022/git_bash_with_pacman_on_windows/#安装-pacman-及其依赖)
- [Index of /msys/x86_64/ (msys2.org)](https://repo.msys2.org/msys/x86_64/)
- [Releases · facebook/zstd (github.com)](https://github.com/facebook/zstd/releases)
- [Windows 的终端配置(给 git-windows 添加 msys2 包管理器) - zeromake 的个人博客](https://blog.zeromake.com/pages/windows-terminal-configuration/)
- [MSYS2 和 mintty 打造 Windows 下 Linux 工具体验 - Creaink - Build something for life](https://creaink.github.io/post/Computer/Windows/win-msys2.html)
- [Using fish shell with git bash on windows (github.com)](https://gist.github.com/rafaelpadovezi/1cfc1026f78255458f5a2ea56291ed23)
- [Package management in Git for Windows (Git Bash)? - Stack Overflow](https://stackoverflow.com/questions/32712133/package-management-in-git-for-windows-git-bash/65204171#comment124229393_65204171)
- [Fish Shell | 安装配置指南 - URmyLucky - 博客园 (cnblogs.com)](https://www.cnblogs.com/Masquer/p/13920104.html)
