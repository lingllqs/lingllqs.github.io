+++
title = 'ArchLinux安装'
date = 2023-10-26T12:08:52+08:00
draft = false
categories = ['教程']
tags = ['ArchLinux']
+++

## 下载 archlinux ISO 镜像

[中科大镜像地址](https://mirrors.ustc.edu.cn/archlinux/iso/)

## 制作启动盘

```bash
windows: 使用 rufus 工具
linux: sudo dd if=/path/to/xxx.iso of=/dev/nvme0n1 bs=4M oflag=sync status=progress
```

## 进入 live 环境开始安装

### 分区

```bash
1. 使用 `fdisk -l` 查看硬盘信息

2. 使用 cfdisk 命令进行分区 `cfdisk /dev/nvme0n1`
这里分两个区
/dev/nvme0n1p1 # efi 引导分区(在 cfdisk 工具界面下方选择 type 进入类型选择界面,选择 efi 类型即可)
/dev/nvme0n1p2 # linux system 分区
```

### 格式化分区

```bash
mkfs.efi -F 32 /dev/nvme0n1p1
mkfs.btrfs -L ARCH /dev/nvme0n1p2 (-L 表示要指定硬盘标签 ARCH 为标签名,可自行命名)
--------------------------------
使用 `lsblk -f` 命令查看分区信息
```

### 创建 btrfs 子卷

```bash
挂载系统分区进行子卷创建
mount --mkdir -t btrfs -o compress=zstd /dev/nvme0n1p2 /tmp/btrfs-root
-----
创建子卷
btrfs subvolume create /tmp/btrfs-root/@     # 根目录
btrfs subvolume create /tmp/btrfs-root/@home # 家目录
btrfs subvolume create /tmp/btrfs-root/@swap # 交换文件
-----
把交换文件子卷写时复制(cow)功能关闭
chattr +C /tmp/btrfs-root/@swap
-----
卸载
umount /tmp/btrfs-root
```

### 挂载分区准备系统安装

```bash
mount --mkdir -t btrfs -o compress=zstd,subvol=@ /dev/nvme0n1p2 /mnt # 注意要先挂载根目录
mount --mkdir -t btrfs -o subvol=@home /dev/nvme0n1p2 /mnt/home      # 挂载根目录下的home目录
mount --mkdir -t btrfs -o subvol=@swap /dev/nvme0n1p2 /mnt/swap
-----------------------------
mount --mkdir /dev/nvme0n1p1 /mnt/boot/efi # 挂载引导分区目录(注意是 nvme0n1p1)
-----------------------------
创建交换文件
btrfs filesystem mkswapfile /mnt/swap/swapfile --uuid clear --size 16G # 大小自行确定
swapon /swap/swapfile # 启用交换文件
```

### 正式安装前先配置好网络

```bash
timedatectl         # 同步系统时间
ip link             # 查看网卡信息
rfkill list         # 查看蓝牙和 wifi 功能是否被关闭
rfkill unblock wlan # 如果 wifi 功能被关闭就启用(或者用`rfkill unblock all`启用所有被关闭功能)
---------------------------------
1. 如果是有线网络则会自动联网无需配置
2. 无线网络则输入 iwctl 命令进入 iw 工具的命令行
device list                  # 列出可用网卡
station wlan0 scan           # 扫描可用网络(wlan0 为网卡名,通过上一步的 `ip link` 命令可知)
station wlan0 get-network    # 获取可用 wifi
station wlan0 connect wifi名 # 连接 wifi，有密码则会提示输入密码
-----
ctrl + d       # 退出 iw 工具命令行
ping -c 3 t.cn # 测试网络是否通畅
```

### 换源

```bash
自动配置: 使用 Python 脚本工具 reflector 配置源
reflector --country China --latest 5 --protocol https --sort rate --save /etc/pacman.d/mirrorlist
----------------------------
手动配置: 编使 vim 工具用辑 /etc/pacman.d/mirrorlist 文件
vim /etc/pacman.d/mirrorlist
在最上面一行添加以下内容:
Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
----------------------------
systemctl stop reflector # 关闭 reflector 服务防止刷新配置
pacman -S archlinux-keyring
pacman -Syy
```

### 开始安装

```bash
pacstrap -K /mnt base linux linux-firmware btrfs-progs # 安装基本的工具，内核和驱动
genfstab -U /mnt >> /mnt/etc/fstab                     # 生成 fstab 文件
arch-chroot /mnt                                       # 进入新系统
-----------------------------------
安装额外的软件和工具
pacman -S base-devel sof-firmware vim grub efibootmgr intel-ucode/amd-ucode firefox networkmanager chcpcd openssh noto-fonts-cjk noto-fonts-emoji nerd-fonts-complete bash-completion
注意: inter-ucode 和 amd-ucode 根据电脑的实际CPU选择安装
```

### 配置

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime                            # 设置时区
hwclock --systohc                                                                  # 将系统时钟同步到硬件时钟
sed -i 's/^#\(en_US\.UTF-8\|zh_CN\.UTF-8\)/\1/g' /etc/locale.gen                   # 设置区域信息
locale-gen && touch /etc/locale.conf && echo "LANG=en_US.UTF-8" > /etc/locale.conf # 生成 locale 信息并设置本地化
touch /etc/hostname && echo "喜欢的主机名" > /etc/hostname                         # 设置主机名
echo -e "127.0.0.1\\tlocalhost\\n::1\\t\\tlocalhost\\n127.0.1.1\\t主机名.domain\\t主机名" >> /etc/hosts
passwd root                             # 设置管理员用户密码
useradd -m -G wheel -s /bin/bash 用户名 # 创建普通用户指定加入 wheel 组
passwd 用户名                           # 给新用户设置密码
编辑 /etc/sudoers 文件把# %wheel ALL=(ALL:ALL) ALL 这一行前面的#去掉
------------------------------------
配置 grub
grub-install --target=x86_64-efi --efi-directory=/boot/efi
grub-mkconfig -o /boot/grub/grub.cfg # 生成 grub 配置
------------------------------------
systemctl enable NetworkManager sshd dhcpcd # 设置一些开机启动的服务
exit 或者 ctrl + d                          # 回到 live 环境
umount -R /mnt                              # 卸载
reboot                                      # 重启
```

## 安装桌面环境或者窗口管理器(Hyprland)

[我的个人Hyprland配置](https://github.com/lingllqs/dotfiles)

