---
title: "Caddy服务器环境配置" # 标题
date: 2023-07-02T17:03:28+08:00	# 创建时间
lastmod: 2023-07-02T17:03:28+08:00	# 更新时间
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
description: "使用Caddy作为服务器环境的配置方法和一些尝试"
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



## 前言 

我接触Caddy是在折腾SteamDeck的时候，因为国内总所周知的网络环境，我不得不使用一些手段确保可以正常登陆、更新Steam，因此想到了使用Caddy进行本地反代的方式，进行Steam的连接。相比另一些插件，Caddy的占用少、轻量和易用的特点深深的吸引了我。因此，我突然想到，如果我使用Caddy作为网页服务器来部署我的博客，那也多是一件美事啊(然后我好不容易搞好的LNMP就炸了)。 由于我太久没有手动安装环境了，已经有点生疏了，因此前前后后折腾了三五天，最终还是实现了我的打算。

由于我写这篇文章的时候，服务器的环境是采用**L**(Linux)**C**(Caddy)**M**(MariaDB)**P**(PHP) + **WordPress**的动态博客，因此环境安装部分也包含了我对于PHP编译和安装数据库的过程，我想既然以前写了，还是保留着吧，毕竟写一篇文章也不容易。。。

## 服务器环境

我的服务器为阿里云轻量应用服务器，配置如下：

| 选项     | 配置   |
| -------- | ------ |
| 性能     | 2C2G   |
| 存储空间 | 60GB   |
| 位置     | 新加坡 |

这样一个低性能的服务器，对于我而言，处于一个可以接受的范围。

一方面，反正也没有多少流量，另一方面，我的支出也不会太过分，每月34元的费用少喝几杯奶茶就有了。此外，非国内的服务器，不需要备案，折腾一些东西（GitHub）也方便许多。

## Caddy安装与配置

### 安装

首先贴上官方中文文档吧（感觉并不是很全，但是够用了）：[官方文档](https://caddy2.dengxiaolong.com/docs/)

我选择的方式是编译安装。

- 首先需要安装**Golang**和**Xcaddy**。
- 克隆Caddy的仓库`git clone "https://github.com/caddyserver/caddy.git"`
- 编译 `xcaddy bulid --with 插件`

### 配置

Caddy的默认配置文件位于`/etc/caddy/Caddyfile`

由于这玩意过于简单明了，我甚至不想分文件来管理Caddy

两个字：**怠惰**

#### 全局配置

```Caddyfile
# 全局配置
{
    email 我的邮箱地址
}
```

#### 日志代码块设置

```Caddyfile
(log) {
    log {
        output file /home/wwwroot/logs/{args.0}/access.log {
            roll_size 100MiB
            roll_local_time
            roll_keep 10
            roll_keep_for 2160h
        }
    }
}
```

#### 反IP扫描

```Caddyfile
:443 {
    abort
}
:80 {
    abort
}
```

#### 站点（动态博客时期）

```Caddyfile
# 个人页配置
me.yblpoi.top {
    encode zstd
    root * /home/wwwroot/vhost/me/
    file_server
    tls /home/wwwroot/cert/yblpoi.top.cer /home/wwwroot/cert/yblpoi.top.key
    import log me
    #Cache
    @cachedFiles {
        path *.webp *.jpg *.jpeg *.png *.gif *.ico *.js *.css *.woff *.woff2 *.ttf
    }
    header @cachedFiles Cache-Control "public, max-age=604800, must-revalidate"
}

# 博客配置
yblpoi.top {
    encode zstd
    root * /home/wwwroot/vhost/blog/
    file_server
    tls /home/wwwroot/cert/yblpoi.top.cer /home/wwwroot/cert/yblpoi.top.key
    php_fastcgi unix//usr/local/php/var/run/php-fpm.sock
    # 引入log
    import log blog
    try_files {path} {path}/ /index.php/?{path}?{query}
 
    #Cache
    @cachedFiles {
        path *.webp *.jpg *.jpeg *.png *.gif *.ico *.js *.css *.woff *.woff2 *.ttf
    }
    header @cachedFiles Cache-Control "public, max-age=604800, must-revalidate"
    #Protect WP Directories
    @disallowed {
        path /xmlrpc.php
        path *.sql
        path "/wp-content/uploads/*.php"
    }
    rewrite @disallowed '/index.php'
}
```

~~***用上静态博客之后我更加怠惰了***~~

#### 防扫ip的说明

在Caddy中，**abort**命令将会直接切断和目标的连接，这个操作甚至是在交换证书之前完成的，我使用Curl在命令行中对我的源站ip进行请求，返回结果如下：

```sh
➜  ~ curl --insecure -v https://myip 
*   Trying myip:443...
* Connected to myip (myip) port 443 (#0)
* ALPN: offers h2,http/1.1
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS alert, internal error (592):
* OpenSSL/3.1.1: error:0A000438:SSL routines::tlsv1 alert internal error
* Closing connection 0
curl: (35) OpenSSL/3.1.1: error:0A000438:SSL routines::tlsv1 alert internal error
➜  ~ curl -v myip
*   Trying myip:80...
* Connected to myip (myip) port 80 (#0)
> GET / HTTP/1.1
> Host: myip
> User-Agent: curl/8.1.2
> Accept: */*
>
* Empty reply from server
* Closing connection 0
curl: (52) Empty reply from server
➜  ~

```

看到这我就放心了。。。

## 附录1·WP的PHP编译配置

编译配置如下：

```sh
 ./configure  --prefix=/usr/local/php --enable-fpm --disable-debug --disable-rpath --enable-shared --with-sqlite3 --with-zlib --enable-bcmath --with-iconv --with-bz2 --with-openssl --enable-calendar --with-curl --with-cdb --enable-dom --enable-exif --enable-fileinfo --enable-filter --with-openssl-dir --with-zlib-dir --enable-gd-jis-conv --with-gettext --with-gmp --with-mhash --enable-mbstring --enable-mbregex --enable-pdo --with-mysqli --with-pdo-mysql --with-pdo-sqlite --with-readline --enable-session --enable-shmop --enable-simplexml --enable-sockets --enable-sysvmsg --enable-sysvsem --enable-sysvshm --with-xsl --enable-mysqlnd-compression-support --with-pear --enable-opcache --with-zip --with-ffi --enable-intl
```

php-fpm的www.conf文件如下：

```ini
[www]
user = caddy
group = caddy
listen = /usr/local/php/var/run/php-fpm.sock
listen.owner = caddy
listen.group = caddy
listen.mode = 0660
pm = dynamic
pm.max_children = 40
pm.start_servers = 10
pm.min_spare_servers = 10
pm.max_spare_servers = 11
```

systemctl的`php-fpm.service`文件内容如下：

```ini
[Unit]
Description=The PHP FastCGI Process Manager
After=network.target
 
[Service]
Type=simple
PIDFile=/usr/local/php/var/run/php-fpm.pid
ExecStart=/usr/local/php/sbin/php-fpm --nodaemonize --fpm-config /usr/local/php/etc/php-fpm.conf
ExecReload=/bin/kill -USR2 $MAINPID
PrivateTmp=true
ProtectSystem=false
PrivateDevices=true
ProtectKernelModules=true
ProtectKernelTunables=true
ProtectControlGroups=true
RestrictRealtime=true
RestrictAddressFamilies=AF_INET AF_INET6 AF_NETLINK AF_UNIX
RestrictNamespaces=true
 
[Install]
WantedBy=multi-user.target
```

## 附录2·常见问题的解决方法

- 检查文件**权限** <u>*(Permission Denied)*</u>
- 检查**配置文件**
- 检查**环境变量**
- 错误日志**Google**之
