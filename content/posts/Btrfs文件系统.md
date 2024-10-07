+++
title = 'Btrfs'
date = 2024-08-27T21:44:30+08:00
draft = false
categories = ['文档']
tags = ['btrfs', '系统快照']
+++


### 给系统做快照

目的: 给根目录('/')做快照

``` shell
    # 硬盘 sda 各个分区挂在情况
    sda
      sda1  /boot
      sda2  /
```

``` shell
    # 硬盘 sdb 各个分区挂在情况
    sdb
      sdb1  /mnt
```

#### 1. 给 sda2 做只读快照(必须是只读快照才能后续的发送 -r 选项)

``` shell
btrfs subvolume snapshot -r / /.newroot
```

#### 2. 发送快照 /.newroot 到 sdb1 上

``` shell
btrfs send /.newroot | btrfs receive /mnt/
```

如果要发送到远程计算机上

``` shell
btrfs send /.newroot | ssh username@remote "btrfs receive /mnt/"
```

#### 3. 在 sdb1 上创建一个普通快照(非只读的)

```shell
btrfs subvolume snapshot /mnt/.newroot /mnt/.curroot
```

#### 4. 把 /mnt/.curroot 快照文件下的内容移动到 sdb1(挂载在/mnt) 下

``` shell
mv /mnt/.curroot/* /mnt/
```

#### 5. 修改 /etc/fstab 文件以便后续系统启动时挂载的根目录是 sdb1

```shell
    - 使用 blkid 命令查看 sdb1 的 UUID
    - 修改 /etc/fstab 里 sda2 的 UUID 为 sdb1 的 UUID
```

#### 6. 修改 /boot/grub/grub.cfg 文件将关于 sda2 的 UUID 都替换为 sdb1 的 UUID

```shell
    - 大概修改行的内容为 root=UUID=xxxxxxxxxxxxxxxxxxxxxxxxx
    - 还有类似的行为 --set=root xxxxxxxxxxxxxxxxxxxxxxxxx
    注意!!! 其中的 xxxxxxxxxxxxxxxxxxxxxxxx 为 sda2 的 UUID 不要把其它的 UUID 改成 sdb1的
```
