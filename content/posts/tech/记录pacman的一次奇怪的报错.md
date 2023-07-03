---
title: "记录pacman的一次奇怪的报错" # 标题
date: 2023-07-03T17:56:28+08:00	# 创建时间
lastmod: 2023-07-03T17:56:28+08:00	# 更新时间
author: ["Yu"]
keywords: 
- Caddy
- 服务器
categories: # 没有分类界面可以不填写
- Caddy
- 服务器环境
tags: # 标签
- Caddy
- 配置
- 云服务器
- Caddyfile
description: "Arch的社区支持真的不错"
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

在我像往常一样，打开电脑，输入`paru`进行更新的时候，突然发现终端有一个报错。

虽然我的电脑一切正常，但是我还是觉得不应该留着这个报错，谁知道留着会带来什么后果呢？

## 报错内容

这次更新包含了`linux-xanmod`内核，报错出现在`mkinitcpio`的过程，所以我怀疑和`mkinitcpio`过程有关。

更新的包：

```shell
:: 正在进行全面系统更新...
正在解析依赖关系...
正在查找软件包冲突...

软件包 (15) evince-44.3-1  haskell-pandoc-server-0.1-76  haskell-servant-server-0.19.2-65  haskell-wai-app-static-3.1.7.4-152  haskell-wai-extra-3.1.13.0-122  haskell-wai-logger-2.4.0-273  haskell-warp-3.3.28-1
            lib32-harfbuzz-7.3.0-2  lib32-icu-73.2-1  lib32-libxml2-2.10.4-5  linux-xanmod-6.4.1-1  ostree-2023.5-1  pandoc-cli-0.1.1-8  python-inflect-6.1.0-1  sdl2-2.28.1-1

下载大小：       91.93 MiB
全部安装大小：  444.18 MiB
净更新大小：      0.84 MiB
```

出现报错位置：

```shell
==> WARNING: Possibly missing firmware for module: 'aic94xx'
==> WARNING: Possibly missing firmware for module: 'bfa'
==> WARNING: Possibly missing firmware for module: 'qed'
==> WARNING: Possibly missing firmware for module: 'qla1280'
==> WARNING: Possibly missing firmware for module: 'qla2xxx'
==> WARNING: Possibly missing firmware for module: 'wd719x'
  -> Running build hook: [filesystems]
  -> Running build hook: [fsck]
==> Generating module dependencies
==> Creating zstd-compressed initcpio image: '/boot/initramfs-linux-xanmod-fallback.img'
==> Image generation successful
错误：命令未能被正确执行
(6/9) Registering Haskell modules...
(7/9) Compiling GSettings XML schema files...
(8/9) Updating icon theme caches...
(9/9) Updating the desktop file MIME type cache...
```

## 问题解决方法

一开始，由于没有具体的报错，也没有出现大的问题，我无法定位到出bug的点。因此我开始尝试搜索有关的报错日志。

皇天不负苦心人，最终找到了出现这个问题的原因（[相关内容：[ SOLVED] initrtamfs error - but no error in messages](https://bbs.archlinux.org/viewtopic.php?id=282234)）：

- 我的文件系统是brtfs
- `btrfs-progs`包并没有安装

由于我的粗心，导致了这一个报错的发生

最后只需要`paru -S btrfs-progs`就解决问题了
