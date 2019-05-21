---
layout:     post
title:      centos
subtitle:   nginx安装 配置ssl
date:       2019-05-14
author:     ZJC
header-img: img/post-bg-debug.png
catalog: true
tags:
    - java
    - 分布式事务
---
在Centos下，yum源不提供nginx的安装，可以通过切换yum源的方法获取安装。也可以通过直接下载安装包的方法，以下命令均需root权限执行：

首先安装必要的库（nginx 中gzip模块需要 zlib 库，rewrite模块需要 pcre 库，ssl 功能需要openssl库）。选定/usr/local为安装目录，以下具体版本号根据实际改变。

# 安装PCRE库

PCRE(Perl Compatible Regular Expressions) 是一个Perl库，包括 perl 兼容的正则表达式库。nginx 的 http 模块使用 pcre 来解析正则表达式，所以需要在 linux 上安装 pcre 库，pcre-devel 是使用 pcre 开发的一个二次开发库。nginx也需要此库。

```
yum -y install pcre-devel

```
# 安装zlib库

zlib 库提供了很多种压缩和解压缩的方式， nginx 使用 zlib 对 http 包的内容进行 gzip ，所以需要在 Centos 上安装 zlib 库。

```
yum -y install pcre-devel

```
# 安装ssl

OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及 SSL 协议，并提供丰富的应用程序供测试或其它目的使用。

nginx 不仅支持 http 协议，还支持 https（即在ssl协议上传输http），所以需要在 Centos 安装 OpenSSL 库。

```
yum install -y openssl openssl-devel

```

# 安装nginx

```
$ cd /usr/local/
$ wget http://nginx.org/download/nginx-1.8.0.tar.gz
$ tar -zxvf nginx-1.8.0.tar.gz
$ cd nginx-1.8.0  
$ ./configure --prefix=/usr/local/nginx --with-http_ssl_module
$ make
$ make install

```

# 启动、停止nginx

```
./nginx 
./nginx -s stop
./nginx -s quit
./nginx -s reload

```

./nginx -t: 查看nginx.conf配置文件是否正确
./nginx -s quit:此方式停止步骤是待nginx进程处理任务完毕进行停止。
./nginx -s stop:此方式相当于先查出nginx进程id再使用kill命令强制杀掉进程。