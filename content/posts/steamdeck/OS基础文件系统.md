---
title: "OS基础·文件系统" # 标题
date: 2023-07-03T23:03:36+08:00	# 创建时间
lastmod: 2023-07-03T23:03:36+08:00	# 更新时间
author: ["Yu"]
keywords: 
- SteamOS
- 文件系统
categories: # 没有分类界面可以不填写
- 
tags: # 标签
- SteamOS
- Linux
- 文件系统
description: "《SteamOS入门到Linux精通》第一讲，SteamOS的文件系统"
weight:
slug: ""
draft: false # 是否为草稿
comments: true # 本页面是否显示评论
reward: true # 打赏
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
    image: "" #图片路径例如：posts/tech/123/123.png
    zoom: # 图片大小，例如填写 50% 表示原图像的一半大小
    caption: "" #图片底部描述
    alt: ""
    relative: false
---





先写一下我对这个系列的吐槽吧：

《SteamOS入门到Linux精通》

思题目学习机，你值得拥有～～

标题是文件系统，但是我其实更想介绍的是**文件系统、分区与各目录**

本章内容主要就是介绍linux与windows的不同，**仅作了解即可**

## 文件系统与分区

Linux系统的文件系统类型和win的完全不同

Linux系统主要以`ext4`、`btrfs`等文件系统为主，而win则是`NTFS`

Linux通过挂载到不同目录的方式，来实现对不同硬盘分区的访问，Windows则是将磁盘挂载为“卷”

### 文件分区与挂载

SteamDeck单系统分区如下：

| 设备名         | 大小 | 类型                 | 作用          |
| -------------- | ---- | -------------------- | ------------- |
| /dev/nvme0n1p1 | 64M  | EFI System           | 引导          |
| /dev/nvme0n1p2 | 32M  | Microsoft Basic DATA | win基础数据   |
| /dev/nvme0n1p3 | 32M  | Microsoft Basic DATA | win基础数据   |
| /dev/nvme0n1p4 | 5G   | Linux root (x86-64)  | Linux根目录   |
| /dev/nvme0n1p5 | 5G   | Linux root (x86-64)  | Linux根目录   |
| /dev/nvme0n1p6 | 256M | Linux variable data  | Linux保留数据 |
| /dev/nvme0n1p7 | 256M | Linux variable data  | Linux保留数据 |
| /dev/nvme0n1p8 | 1.9T | Linux home           | 用户目录      |

SteamDeck单系统`fstab`文件挂载选项

```fstab
# Static information about the filesystems.
# See fstab(5) for details.

# <file system> <dir> <type> <options> <dump> <pass>
# SteamOS partitions
#/dev/disk/by-partsets/self/rootfs /       ext4    defaults                0       1
#/dev/disk/by-partsets/self/var    /var    ext4    defaults                0       2
/dev/disk/by-partsets/self/efi    /efi    vfat    defaults,nofail,umask=0077,x-systemd.automount,x-systemd.idle-timeout=1min 0       2
/dev/disk/by-partsets/shared/esp  /esp    vfat    defaults,nofail,umask=0077,x-systemd.automount,x-systemd.idle-timeout=1min 0       2
/dev/disk/by-partsets/shared/home /home   ext4    defaults,nofail,x-systemd.growfs 0       2

```

从fstab挂载文件来看，我们日常使用的`/home`目录为`ext4`格式，从分区表可得，挂载的磁盘分区为`/dev/nvme0n1p8`

至于为什么有很多相同类型的分区。。。我的**猜测**是，这相当于是一个备份分区，当需要还原系统时，使用另一个分区的文件覆盖掉正在使用的分区即可。

同时介绍一下**ext4**这个文件格式吧

> Ext4 是最常用的 Linux 文件系统 Ext3 的进化。在许多方面，Ext4对于 Ext3 有着比 Ext3 对于 Ext2 更多更深的改变。Ext3 主要是向 Ext2 添加了日志系统，而 Ext4 修改了重要的文件系统的数据结构，比如用来存储文件数据的那部分。当然结果就是文件系统有更好的设计，更好的性能，稳定性还有更多的功能。
>
> 作为一个先进的文件系统，ext4不需要像NTFS一样需要4K对齐，它会自动进行4K对齐。
>
> ext4 使用 48 位的内部寻址，理论上可以在文件系统上分配高达 16 TiB 大小的文件，其中文件系统大小最高可达 1000000 TiB（1 EiB）。

是不是感觉Linux瞬间🐮了起来。

## 文件类型

在linux中，有四种基本文件类型，分别为

1. 普通文件
2. 目录文件
3. 链接文件
4. 特殊文件

### 普通文件

我们平时使用的增删改查运行就是针对普通文件的。下面列举一些：

- 文本文件（包括各种源代码、配置文件）
- 二进制可执行文件（类似于Windows中的EXE文件）
- shell脚本

### 目录文件

包括文件名、子目录名及其指针。

查看目录可以使用`ls`命令

### 链接文件

类似于windows中的快捷方式，使用`ln`命令创建

链接文件可以指向某一个文件，也可以指向某个目录。

主要分为软链接和硬连接两种

### 特殊文件

在linux中，一切皆文件。因此CPU、GPU、内存等硬件也被具象化为一个个文件显示了出来。并且可以通过各种命令修改其配置、读取其状态。

比如上面硬盘分区的`/dev/nvme0n1`就是第一个固态硬盘在linux中的文件显示。

## 目录

通过命令，显示根目录，可以看到Linux的重要目录：

```shell
/
├── bin -> usr/bin
├── boot
├── dev
├── etc
├── home
├── lib -> usr/lib
├── lib64 -> usr/lib
├── mnt
├── opt
├── proc
├── root
├── run
├── sbin -> usr/bin
├── srv
├── sys
├── tmp
├── usr
└── var
```

### 整体介绍

简单介绍一下吧：

- **/bin**：这个目录包含了系统中最基本的命令，如ls、cp、mv等，这些命令在系统启动时就可用。
- **/boot**：包含启动Linux系统所需的相关文件，包括内核（kernel）和引导加载器（bootloader）。
- **/dev**：包含设备文件，Linux中设备也被视为文件，这个目录包含了所有硬件设备和外部设备的访问点。
- **/etc**：这个目录包含系统的配置文件，例如网络配置、用户账户信息、服务配置等。
- **/home**：普通用户的家目录，每个用户都有一个自己的子目录在这里。
- **/lib**：这个目录包含着许多共享库文件，这些库文件被程序在运行时共享使用。
- **/media**：用于挂载可移动媒体设备，例如光盘、USB闪存驱动器等。
- **/mnt**：通常用于临时挂载其他文件系统或网络共享。
- **/opt**：这个目录通常用于安装额外的软件包（非系统默认提供的），各个软件包通常会有自己的子目录。
- **/proc**：虚拟文件系统，提供有关内核和运行中进程的信息。
- **/root**：超级用户（root）的家目录。
- **/run**：存储系统运行时的临时文件，例如PID文件和socket文件。
- **/sbin**：与/bin类似，包含系统管理命令，但这些命令通常只有超级用户可以运行。
- **/srv**：用于存储特定服务提供的数据，例如网站数据。
- **/sys**：也是一个虚拟文件系统，用于访问内核相关信息。
- **/tmp**：用于存储临时文件的目录，系统重启时会自动清空其中的内容。
- **/usr**：包含用户安装的应用程序和文件，类似于Program Files目录。
- **/var**：包含经常变化的文件，如日志文件、缓存文件、邮件等。

### 常用目录

其实用的最多的应该就2～3个目录。如果只是为了玩游戏、没有折腾生产力或者开源软件的需求，甚至只会接触1个目录。

- **/home/deck**:这是Steamdeck默认用户的目录，所有的游戏都是存在于此目录。（deck用户下方便的表达方式为**''~/''**）
  - **~/.config**: 用户配置目录
  - **~/.ssh**: ssh公私钥存放目录。（权限一章会详细介绍）
  - **~/.steam**: Steam游戏、MOD、自定义兼容层所在目录
  - **~/.cache**: 顾名思义，缓存目录
- **/etc**: 如上文所说，包含系统配置文件等。此目录也较为常用
- **/boot**: 折腾双系统会用到
- **/usr**: 如果你想编译安装一些软件的话，应该用的上。。。

****

**[点我返回目录](https://yblpoi.top/posts/steamdeck/steamdeck%E5%AE%8C%E5%85%A8%E6%8A%98%E8%85%BE%E6%8C%87%E5%8D%97%E5%BC%80%E7%AF%87/)**
