# Debian配置Btrfs文件系统


<!--more-->

# Debian Sid配置Btrfs文件系统

## 安装Debian Testing

1. 安装时选择Advanced options -> Expert install
> 当提示“设置用户和密码”时，不允许以 root 身份登录。这样我们的用户将被自动设置在“sudo”组中。
2. 其他选择和正常安装一样，直到到达Partition disks(磁盘分区)界面，选择手动分区
3. 选择gpt
4. 创建一个EFI分区，大小为512M，文件系统为EFI System Partition
5. 创建一个/boot分区，大小为1024M，文件系统为ext4（如果需要加密则创建此分区）
6. 剩下的空间创建一个根分区，文件系统为btrfs，分区完成。
> 当询问是否要返回并创建 SWAP 分区时，选择“否”，我们不需要磁盘上的 SWAP 分区，因为我们将使用 ZRAM 进行 SWAP
7. 在选择Install the base system之前，按alt+ctrl+F2(笔记本可能是： alt+ctrl+fn+F2)切换到tty2，执行以下命令
```bash
df

umount /target/boot/efi/
# 可选
umount /target/boot/
umount /target/

# 根据df命令选择对应的分区
mount /dev/sda2 /mnt

cd /mnt
mv @rootfs @

# 创建其他子卷
btrfs su cr @snapshots
btrfs su cr @home
btrfs su cr @opt
btrfs su cr @log
btrfs su cr @cache
btrfs su cr @crash
btrfs su cr @tmp
btrfs su cr @spool

btrfs su cr @images
btrfs su cr @containers
# GNOME related subvolumes，如果不使用GNOME则不需要，在安装gnome桌面前不能挂载此2个子卷，否则会出现fstab挂载错误
btrfs su cr @AccountsService
btrfs su cr @gdm

# 挂载子卷
mount -o noatime,space_cache=v2,compress=zstd:1,ssd,discard=async,subvol=@ /dev/mapper/sda2 /target

# 创建其他子卷挂载目录
cd /target
mkdir -p .snapshots
mkdir -p home
mkdir -p opt
mkdir -p var/log
mkdir -p var/cache
mkdir -p var/crash
mkdir -p var/tmp
mkdir -p var/spool
mkdir -p var/lib/libvirt/images
mkdir -p var/lib/containers
# 如果不安装GNOME则不需要此步骤
mkdir -p var/lib/AccountsService
mkdir -p var/lib/gdm3

# 挂载子卷到指定目录
mount -o noatime,space_cache=v2,compress=zstd:1,ssd,discard=async,subvol=@snapshots /dev/mapper/sda2 /target/.snapshots
mount -o noatime,space_cache=v2,compress=zstd:1,ssd,discard=async,subvol=@home /dev/mapper/sda2 /target/home
mount -o noatime,space_cache=v2,compress=zstd:1,ssd,discard=async,subvol=@home /dev/mapper/sda2 /target/opt
mount -o noatime,space_cache=v2,compress=zstd:1,ssd,discard=async,subvol=@log /dev/mapper/sda2 /target/var/log
mount -o noatime,space_cache=v2,compress=zstd:1,ssd,discard=async,subvol=@cache /dev/mapper/sda2 /target/var/cache
mount -o noatime,space_cache=v2,compress=zstd:1,ssd,discard=async,subvol=@crash /dev/mapper/sda2 /target/var/crash
mount -o noatime,space_cache=v2,compress=zstd:1,ssd,discard=async,subvol=@tmp /dev/mapper/sda2 /target/var/tmp
mount -o noatime,space_cache=v2,compress=zstd:1,ssd,discard=async,subvol=@spool /dev/mapper/sda2 /target/var/spool
mount -o noatime,space_cache=v2,compress=zstd:1,ssd,discard=async,subvol=@images /dev/mapper/sda2 /target/var/lib/libvirt/images
mount -o noatime,space_cache=v2,compress=zstd:1,ssd,discard=async,subvol=@containers /dev/mapper/sda2 /target/var/lib/containers
# 下面两个步骤先不执行，等安装GNOME桌面后再执行
# mount -o noatime,space_cache=v2,compress=zstd:1,ssd,discard=async,subvol=@AccountsService /dev/mapper/sda2 /target/var/lib/AccountsService
# mount -o noatime,space_cache=v2,compress=zstd:1,ssd,discard=async,subvol=@gdm /dev/mapper/sda2 /target/var/lib/gdm3

# 再挂载BOOT分区，EFI分区
# mount /dev/sda2 boot
mount /dev/sda1 boot/efi

# 持久化挂载
nano etc/fstab
/dev/mapper/$VOLUME_GROUP /                        btrfs  noatime,space_cache=v2,compress=zstd:1,ssd,discard=async,subvol=@                   0    0
/dev/mapper/$VOLUME_GROUP /.snapshots              btrfs  noatime,space_cache=v2,compress=zstd:1,ssd,discard=async,subvol=@snapshots          0    0
/dev/mapper/$VOLUME_GROUP /home                    btrfs  noatime,space_cache=v2,compress=zstd:1,ssd,discard=async,subvol=@home               0    0
/dev/mapper/$VOLUME_GROUP /opt                     btrfs  noatime,space_cache=v2,compress=zstd:1,ssd,discard=async,subvol=@opt                0    0
/dev/mapper/$VOLUME_GROUP /var/log                 btrfs  noatime,space_cache=v2,compress=zstd:1,ssd,discard=async,subvol=@log                0    0
/dev/mapper/$VOLUME_GROUP /var/cache               btrfs  noatime,space_cache=v2,compress=zstd:1,ssd,discard=async,subvol=@cache              0    0
/dev/mapper/$VOLUME_GROUP /var/crash               btrfs  noatime,space_cache=v2,compress=zstd:1,ssd,discard=async,subvol=@crash              0    0
/dev/mapper/$VOLUME_GROUP /var/tmp                 btrfs  noatime,space_cache=v2,compress=zstd:1,ssd,discard=async,subvol=@tmp                0    0
/dev/mapper/$VOLUME_GROUP /var/spool               btrfs  noatime,space_cache=v2,compress=zstd:1,ssd,discard=async,subvol=@spool              0    0
/dev/mapper/$VOLUME_GROUP /var/lib/libvirt/images  btrfs  noatime,space_cache=v2,compress=zstd:1,ssd,discard=async,subvol=@images             0    0
/dev/mapper/$VOLUME_GROUP /var/lib/containers      btrfs  noatime,space_cache=v2,compress=zstd:1,ssd,discard=async,subvol=@containers         0    0
# /dev/mapper/$VOLUME_GROUP /var/lib/AccountsService btrfs  noatime,space_cache=v2,compress=zstd:1,ssd,discard=async,subvol=@AccountsService    0    0
# /dev/mapper/$VOLUME_GROUP /var/lib/gdm3            btrfs  noatime,space_cache=v2,compress=zstd:1,ssd,discard=async,subvol=@gdm                0    0

# 卸载挂载
cd /
umount /mnt
exit
```
8. 按ctrl+alt+F1(笔记本可能是ctrl+alt+Fn+F1)回到tty1，选择Install the base system
9. 软件选择时，取消基本桌面环境,Gnome,只选择"standard system utilities" 即可
10. 其他步骤按照正常执行即可,最后重启系统执行以下步骤

## 配置ZRAM
**执行命令安装zram**
```bash
sudo apt update && sudo apt upgrade
sudo apt install zram-tools
```

更改配置文件`sudo nano /etc/default/zramswap`将`ALGORITHM`改为zstd，将`SIZE`改为8192(即8GB)的内存大小


## 配置Snapper
**安装工具**
```bash
sudo apt install snapper inotify-tools git make
```
**执行配置**
```bash
# 因为snapper创建配置时会在当前执行目录下面一个.snapshots子卷和.snapshots目录，而我们希望将/.snapshots挂载到@shapshots子卷，所以需要先卸载@shapshots子卷
cd
sudo umount /.snapshots
sudo rm -rf .snapshots

# 创建配置
sudo snapper -c root create-config /

# 删除snapper自己创建/.snapshots子卷,并将/.snapshots挂载到@snapshots子卷
sudo btrfs subvolume delete /.snapshots
sudo mkdir /.snapshots
sudo mount -av
```

**配置Snapper自动快照**

1. 关闭开机快照
```bash
sudo systemctl disable snapper-boot.timer
```
2. 关闭timeline快照
```bash
sudo snapper -c root set-config 'TIMELINE_CREATE=no'
```

3. 开启apt pre/post快照,并优化打印信息
snapper会自动开启apt pre/post快照，但是默认打印命令不够详细，可以通过以下命令优化

- 3.1 修改`/etc/apt/apt.conf.d/80snapper`文件内容为
```bash
# https://gist.github.com/imthenachoman/f722f6d08dfb404fed2a3b2d83263118
# https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=770938

DPkg::Pre-Invoke { "/path/to/dpkg-pre-post-snapper.sh pre"; };
DPkg::Post-Invoke { "/path/to/dpkg-pre-post-snapper.sh post"; };
```

- 3.2 创建`/path/to/dpkg-pre-post-snapper.sh`文件内容为
```bash
#!/bin/bash

# this script is an enhancement of https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=770938

# we need to work up the process tree to find the apt command that triggered the call to this script
# get the initial PID

PID=$$

# find the apt command by working up the process tree
# loop until
# - PID is empty
# - PID is 1
# - or PID command is apt
while [[ -n "$PID" && "$PID" != "1" && "$(ps -ho comm "${PID}")" != "apt" ]] ; do
    # the current PID is not the apt command so go up one by getting the parent PID of hte current PID
    PID=$(ps -ho ppid "$PID" | xargs)
done

SNAPPER_DESCRIPTION="apt"

# assuming we found the apt command, get the full args
if [[ "$(ps -ho comm "${PID}")" = "apt" ]] ; then
    SNAPPER_DESCRIPTION="$(ps -ho args "${PID}")"
fi

# main event

# source /etc/default/snapper if it exists
if [ -e /etc/default/snapper ] ; then
    . /etc/default/snapper
fi

# what action are we taking
if [ "$1" = "pre" ] ; then
    # pre, so take a pre snapshot

    # if snapper is installed
    # and if snapper snapshots are not being disabled using the DISABLE_APT_SNAPSHOT variable
    # and if /etc/snapper/configs/root exists
    if [ -x /usr/bin/snapper ] && [ ! x$DISABLE_APT_SNAPSHOT = 'xyes' ] && [ -e /etc/snapper/configs/root ] ; then
        # delete any lingering temp files
        rm -f /var/tmp/snapper-apt || true

        # create a snapshot
        # and save the snapshot number for reference later
        snapper create -d "${SNAPPER_DESCRIPTION}" -c number -t pre -p > /var/tmp/snapper-apt || true

        # clean up snapper
        snapper cleanup number || true
    fi
elif [ "$1" = "post" ] ; then
    # post, so take a post snapshot

    # if snapper is installed
    # and if snapper snapshots are not being disabled using the DISABLE_APT_SNAPSHOT variable
    # and if the temp file with the snapshot number from the pre snapshot exists
    if [ -x /usr/bin/snapper ] && [ ! x$DISABLE_APT_SNAPSHOT = 'xyes' ] && [ -e /var/tmp/snapper-apt ]
    then
        # take a post snapshot and link it to the # of the pre snapshot
        snapper create -d "${SNAPPER_DESCRIPTION}" -c number -t post --pre-number=`cat /var/tmp/snapper-apt` || true

        # clean up snapper
        snapper cleanup number || true
    fi
fi
```

4. 配置sudo使用snapper使用权限
```bash
sudo snapper -c root set-config 'ALLOW_GROUPS=sudo' 'SYNC_ACL=yes'
```

5. 配置GRUB-BTRFS

- 5.1 安装
GRUB-BTRFS在Debian上不可用,所以需要编译安装
```bash
cd
git clone https://github.com/Antynea/grub-btrfs.git
cd grub-btrfs
sudo make install
```

- 5.2 配置
```bash
# To start the daemon run:
sudo systemctl start grub-btrfsd
# To activate it during system startup, run:
sudo systemctl enable grub-btrfsd
```

- 5.3 下载的grub-btrfs代码可以清除了
```bash
cd ..
sudo rm -rf grub-btrfs/
```
> grub-btrfs可以在GRUB中显示当前系统中的所有快照,如果系统出问题了,可以从grub选择之前正常的快照回滚到可以正常启动的系统

## 安装Gnome桌面
```bash
sudo apt install gnome-core
```

---

> 作者: map[avatar:<nil> email:<nil> link:<nil> name:<nil>]  
> URL: https://blog-12x.pages.dev/debian%E9%85%8D%E7%BD%AEbtrfs%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F/  

