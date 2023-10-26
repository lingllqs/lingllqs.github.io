+++
title = 'ArchLinux安装'
date = 2023-10-26T12:08:52+08:00
draft = false
categories = ['教程']
tags = ['ArchLinux']
+++

## 下载 archlinux 镜像


## 制作启动盘

```bash
sudo dd if=/path/to/xxx.iso of=/dev/nvme0n1 bs=4M oflag=sync status=progress
```

## 进入 live 环境开始安装

### 分区

```bash
cfdisk /dev/nvme0n1
分两个区
/dev/nvme0n1p1 # efi 分区
/dev/nvme0n1p2 # linux system 分区
```

### 格式化分区

```bash
mkfs.efi -F 32 /dev/nvme0n1p1
mkfs.btrfs -L ARCH /dev/nvme0n1p2
-----
使用 `lsblk -f` 命令查看分区信息
```

### 创建 btrfs 子卷

```bash
挂载系统分区
mount --mkdir -t btrfs -o compress=zstd /dev/nvme0n1p2 /tmp/btrfs-root
-----
创建子卷
btrfs subvolume create /dev/nvme0n1p2 /tmp/btrfs-root/@     # 根目录
btrfs subvolume create /dev/nvme0n1p2 /tmp/btrfs-root/@home # 家目录
btrfs subvolume create /dev/nvme0n1p2 /tmp/btrfs-root/@swap # 交换文件
-----
把交换文件子卷写时复制(cow)功能关闭
chattr +C /tmp/btrfs-root/@swap
-----
卸载
umount /tmp/btrfs-root
```

### 挂载分区准备安装

```bash
mount --mkdir /dev/nvme0n1p1 /mnt/boot/efi
-----
mount --mkdir -t btrfs -o compress=zstd,subvol=@ /dev/nvme0n1p2 /mnt
mount --mkdir -t btrfs -o subvol=@home /dev/nvme0n1p2 /mnt/home
mount --mkdir -t btrfs -o subvol=@swap /dev/nvme0n1p2 /mnt/swap
-----
创建交换文件
btrfs filesystem mkswapfile /mnt/swap/swapfile --uuid clear --size 16G # 大小自行确定
```

### 正式安装前先配置好网络

```bash
ip link             # 查看网卡信息
rfkill list         # 查看蓝牙和 wifi 功能是否被关闭
rfkill unblock wlan # 如果 wifi 功能被关闭就启用
-----
输入 iwctl 命令进入 iw 工具的命令行
device list                           # 列出可用网卡
station wlan0 scan                    # 扫描可用网络
station wlan0 get-network             # 获取可用 wifi
station wlan0 connect wifi名 wifi密码 # 连接 wifi，没有密码可以不用输入
-----
ctrl + d       # 退出 iw 工具命令行
ping -c 3 t.cn # 测试网络是否通畅
```

### 换源

```bash
自动配置: 使用 Python 脚本工具 reflector 配置源
reflector --country China --latest 5 --protocol https --sort rate --save /etc/pacman.d/mirrorlist
systemctl stop reflector # 关闭 reflector 服务放置刷新配置
-----
手动配置: 编辑 /etc/pacman.d/mirrorlist 文件
vim /etc/pacman.d/mirrorlist
在最上面一行添加以下内容:
Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
```
