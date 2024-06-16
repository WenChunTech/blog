---
title: ArchLinux安装指导
subtitle:
date: 2024-06-15T13:28:03+08:00
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
  - linux
  - arch
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

# ArchLinux安装指导

##  1. 下载镜像

	- [中国科学技术大学开源镜像站](http://mirrors.ustc.edu.cn/)

	- [清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/)

	- [华为开源镜像站](https://repo.huaweicloud.com/archlinux/)

	- [兰州大学开源镜像站](https://mirror.lzu.edu.cn/archlinux/)

## 2. 制作安装盘

Windows 下推荐使用 [balenaEtcher](https://www.balena.io/etcher/)，[Ventoy](https://www.ventoy.net/cn/doc_start.html)，[Rufus](https://rufus.ie/) 或者 [Power ISO](https://www.poweriso.com/download.php)EULA 进行 U 盘刻录。

## 2.（可选）如果需要安装双系统，需要关闭或者获取Bitlocker恢复密钥

## 3.  关闭 BIOS 设置中的 Secure Boot

## 4. 调整启动方式为 UEFI（可能不需要）

## 5. 设置U盘为首启动

## 6. 基础安装

### 6.1  禁用 reflector 服务

1. 通过以下命令将该服务禁用：

```bash
systemctl stop reflector.service
```

通过以下命令查看该服务是否被禁用，按下 `q` 退出结果输出：

```bash
systemctl status reflector.service
```

### 6.2. 再次确认是否为 UEFI 模式

禁用 `reflector` 服务后，我们再来确认一下是否为 `UEFI` 模式：

```bash
ls /sys/firmware/efi/efivars
```

若输出了一堆东西（`efi` 变量），则说明已在 `UEFI` 模式。否则请确认你的启动方式是否为 `UEFI`。

### 6.3 连接无线网络

使用 `iwctl` 进行连接：

```bash
iwctl # 进入交互式命令行
device list # 列出无线网卡设备名，比如无线网卡看到叫 wlan0
station wlan0 scan # 扫描网络
station wlan0 get-networks # 列出所有 wifi 网络
station wlan0 connect wifi-name # 进行连接，注意这里无法输入中文。回车后输入密码即可
exit # 连接成功后退出
```

### 6.4 更新系统时钟

使用 `timedatectl` 确保系统时间是准确的。这一步**不是**可选的，正确的系统时间对于部分程序来说非常重要：

```bash
timedatectl set-ntp true # 将系统时间与网络时间进行同步
timedatectl status # 检查服务状态
```

### 6.5 更换国内镜像源

修改 `/etc/pacman.d/mirrorlist`放在最上面的是会使用的软件仓库镜像源，推荐的镜像源如下：

```bash
Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch # 中国科学技术大学开源镜像站
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch # 清华大学开源软件镜像站
Server = https://repo.huaweicloud.com/archlinux/$repo/os/$arch # 华为开源镜像站
Server = http://mirror.lzu.edu.cn/archlinux/$repo/os/$arch # 兰州大学开源镜像站
```

> 注意： 请不在这一步中添加 `archlinuxcn` 源

### 6.6  分区

#### 6.6.1 分区表：

| 挂载目录  | 文件类型                          | 分配大小           |
| --------- | --------------------------------- | ------------------ |
| /boot/efi | EFI分区                           | 1G                 |
| swap      | Linux swap（Swap分区）            | 一般和内存大小一致 |
| /         | Linux filesystem（Btrfs文件系统） | 剩余大小           |

```bash
lsblk # 查看分区情况
cfdisk /dev/sdx # 对系统进行分区
fdisk -l # 查看分区情况
```

#### 6.6.2 格式化分区

```bash
# EFI 分区格式化
mkfs.fat -F32 /dev/sdxn

# Swap 分区格式化
mkswap /dev/sdxn
# Btrfs 根目录分区格式化
mkfs.btrfs -L Arch /dev/sdxn
```

6.6.3 创建子卷

```bash
# Btrfs 分区挂载到 /mnt 下
mount -t btrfs -o compress=zstd /dev/sdxn /mnt

df -h # 查看挂载情况

# 创建子卷
btrfs subvolume create /mnt/@ # 创建 / 目录子卷
btrfs subvolume create /mnt/@home # 创建 /home 目录子卷
btrfs subvolume create /mnt/@opt # 创建 /opt 目录子卷
# 如果有创建swap分区，就不需要创建这个@swap子卷了，我使用swapfile所以创建了一个子卷
btrfs subvolume create /mnt/@swap # 创建 /swap 目录子卷
btrfs subvolume create /mnt/@var_log # 创建 /var/log 目录子卷
btrfs subvolume create /mnt/@var_tmp # 创建 /var/tmp 目录子卷
btrfs subvolume create /mnt/@var_crash # 创建 /var/crash 目录子卷
btrfs subvolume create /mnt/@var_cache # 创建 /var/cache 目录子卷
btrfs subvolume create /mnt/@pkg # 创建 /var/cache/pacman/pkg 目录子卷

#或者使用以下命令全部创建
cd /mnt
mkdir @ @home @opt @swap @var_log @var_tmp @var_crash @var_cache @pkg

# 查看子卷创建情况
btrfs subvolume list -p /mnt

# 卸载/mnt, 挂载子卷
cd /
umount /mnt

# 挂载 / 目录
mount -t btrfs -o subvol=/@,compress=zstd /dev/sdxn /mnt
# 创建子卷对应的目录
cd /mnt
mkdir -p home swap opt var/log var/tmp var/crash var/cache var/cache/pacman/pkg

mount -t btrfs -o subvol=/@home,compress=zstd /dev/sdxn /mnt/home
mount -t btrfs -o subvol=/@opt,compress=zstd /dev/sdxn /mnt/opt
mount -t btrfs -o subvol=/@swap,compress=zstd /dev/sdxn /mnt/swap
mount -t btrfs -o subvol=/@var_log,compress=zstd /dev/sdxn /mnt/var/log
mount -t btrfs -o subvol=/@var_tmp,compress=zstd /dev/sdxn /mnt/var/tmp
mount -t btrfs -o subvol=/@var_cache,compress=zstd /dev/sdxn /mnt/var/cache
mount -t btrfs -o subvol=/@pkg,compress=zstd /dev/sdxn /mnt/var/cache/pacman/pkg

# 挂载 /boot 目录
mkdir -p /mnt/boot
mount /dev/sdxn /mnt/boot

# 挂载交换分区
swapon /dev/sdxn

# 查看挂载情况
df -h

# 查看swap分区挂载情况
free -h
```



## 7. 安装系统

### 7.1 使用 `pacstrap` 脚本安装基础包

```bash
pacstrap /mnt base base-devel linux-zen linux-zen-firmware btrfs-progs networkmanager nvim sudo zsh zsh-completions timeshift
```

> - `base-devel` —— `base-devel` 在 `AUR` 包的安装过程中是必须用到的
> - `linux` —— 内核软件包

### 7.2 生成fstab文件

`fstab` 用来定义磁盘分区。它是 Linux 系统中重要的文件之一。使用 `genfstab` 自动根据当前挂载情况生成并写入 `fstab` 文件：

```bash
genfstab -U /mnt > /mnt/etc/fstab

# 查看fstab文件有没有问题
cat /mnt/etc/fstab
```

### 7.3 change root

```
arch-chroot /mnt
```

### 7.4 一些必要设置

```bash
# 设置主机名
vim /etc/hostname

# 修改hosts, 修改/etc/hosts为以下内容
127.0.0.1   localhost
::1         localhost
127.0.1.1   myarch.localdomain myarch

# 设置时区
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

# 同步系统时间到硬件时间
hwclock --systohc

# 设置locale，编辑 /etc/locale.gen，去掉 en_US.UTF-8 UTF-8 以及 zh_CN.UTF-8 UTF-8 行前的注释符号（#）
vim /etc/locale.gen

# 生成locale
locale-gen

# 向 /etc/locale.conf 输入内容， 不推荐在此设置任何中文 locale，会导致 tty 乱码。
echo 'LANG=en_US.UTF-8'  > /etc/locale.conf

# 设置root密码
passwd root

# 安装微码
pacman -S intel-ucode # Intel
pacman -S amd-ucode # AMD

# 安装引导程序，grub: 启动引导器，efibootmgr：efibootmgr 被 grub 脚本用来将启动项写入 NVRAM， os-prober：为了能够引导 win10，需要安装 os-prober 以检测到它
pacman -S grub efibootmgr os-prober

# 安装Grub到EFI分区
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=ARCH

# （可选） 编辑 /etc/default/grub 文件，
# 去掉 GRUB_CMDLINE_LINUX_DEFAULT 一行中最后的 quiet 参数
# 把 loglevel 的数值从 3 改成 5。这样是为了后续如果出现系统错误，方便排错
# 加入 nowatchdog 参数，这可以显著提高开关机速度
# 如果引导 Windows，则还需要添加新的一行 GRUB_DISABLE_OS_PROBER=false
vim /etc/default/grub

# 生成Grub所需的配置文件
grub-mkconfig -o /boot/grub/grub.cfg

# 完成安装
exit # 退回安装环境
umount -R /mnt # 卸载新分区
reboot # 重启
```

### 7.5 安装重启后的一些配置

```bash
# 设置开机自启并立即启动 NetworkManager 服务
systemctl enable --now NetworkManager
# 测试网络连接
ping www.bilibili.com

# 无线连接配置
nmcli dev wifi list # 显示附近的 Wi-Fi 网络
nmcli dev wifi connect "Wi-Fi名（SSID）" password "网络密码" # 连接指定的无线网络

# 也可以使用nmtui配置网络
nmtui

# 更新到最新系统
pacman -Syu

# 配置root账户默认编辑器：编辑`~/.bash_profile`文件
vim ~/.bash_profile
# 加入以下内容
export EDITOR='nvim'

# 添加非root用户，并设置密码
useradd -m -G wheel -s /bin/bash myusername
passwd myusername

# 通过 visudo 命令编辑 sudoers 文件
visudo
# 到如下这样的一行，把前面的注释符号 # 去掉：
#%wheel ALL=(ALL:ALL) ALL

# 开启 32 位支持库与 Arch Linux 中文社区仓库
vim /etc/pacman.conf
# 去掉 [multilib] 一节中两行的注释，来开启 32 位库支持
# 在文档结尾处加入下面的文字，来添加 archlinuxcn 源。推荐的镜像源（选一个即可）也一并列出
[archlinuxcn]
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch # 中国科学技术大学开源镜像站
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch # 清华大学开源软件镜像站
Server = https://mirrors.hit.edu.cn/archlinuxcn/$arch # 哈尔滨工业大学开源镜像站
Server = https://repo.huaweicloud.com/archlinuxcn/$arch # 华为开源镜像站

# 刷新 pacman 数据库并更新
pacman -Syyu

# 配置 swapfile 文件，使用btrfs文件系统
btrfs filesystem mkswapfile --size 4g --uuid clear /swap/swapfile
# 启用交换文件
swapon /swap/swapfile
# 编辑fstab，添加交换文件的配置
/swap/swapfile none swap defaults 0 0

# 配置休眠
# 配置 initramfs，编辑/etc/mkinitcpio.conf文件，需要在udev钩子之后加入resume 钩子，例如：
HOOKS=(base udev autodetect keyboard modconf block filesystems resume fsck)
# 传入内核参数，因为我使用的是swapfile，而不是swap分区需要额外配置 resume offset
# resume offset 值获取
btrfs inspect-internal map-swapfile -r path/to/swapfile
# UUID参数可以从fstab文件，或者blkid命令查询
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash loglevel=3 rd.udev.log_priority=3 vt.global_cursor_default=0 resume=UUID=039afb4e-b97a-4725-ae2e-993c8a9ff609 resume_offset=3451070"
# 最后再让配置生效
mkinitcpio -P linux-zen
grub-mkconfig -o /boot/grub/grub.cfg

# 做一次基础安装快照

```

- [电源管理/挂起与休眠 - Arch Linux 中文维基 (archlinuxcn.org)](https://wiki.archlinuxcn.org/wiki/电源管理/挂起与休眠)
- [ArchLinux 休眠到交换文件 | Harttle Land](https://harttle.land/2019/10/19/hibernate-archlinux.html)
- [现代化的 Archlinux 安装，Btrfs、快照、休眠以及更多。 - 少数派 (sspai.com)](https://sspai.com/post/78916)
- [archlinux 基础安装 | archlinux 简明指南 (icekylin.online)](https://arch.icekylin.online/guide/rookie/basic-install.html)

