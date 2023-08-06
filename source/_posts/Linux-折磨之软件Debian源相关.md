---
title: Linux-折磨之软件Debian源相关
top: false
cover: false
toc: true
mathjax: true
categories:
  - linux
  - 软件源
tags:
  - apt
  - linux
date: 2023-07-13 23:33:57
password:
summary:
---

# Debian 9 软件源问题

## 背景

> docker下容器给予debian 9想下载软件包，无奈怎么弄都是失败，apt-update各种报错

## 查看系统版本

```bash
root@4965c2bf1d67:/# cat /etc/os-release
PRETTY_NAME="Debian GNU/Linux 9 (stretch)"
NAME="Debian GNU/Linux"
VERSION_ID="9"
VERSION="9 (stretch)"
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
root@4965c2bf1d67:/#
```

## 换源

> 从2023年4月23日起，debian9的源包地址更换至新地址。 新地址如下：

```bash
deb http://archive.debian.org/debian/ stretch main contrib non-free
deb-src http://archive.debian.org/debian/ stretch main contrib non-free
deb http://archive.debian.org/debian-security/ stretch/updates main contrib non-free
deb-src http://archive.debian.org/debian-security/ stretch/updates main contrib non-free
deb http://archive.debian.org/debian/ stretch-backports main contrib non-free
```

将上面内容替换到`/etc/apt/sources.list`中，然后执行`sudo apt-get update`即可。

[docker debian源个别依赖包无法下载 #1547](https://github.com/tuna/issues/issues/1547)

[Debian换apt](https://www.ztrl.tech/38.html)

[CentOS7 安装源更换为腾讯源：](https://www.rxinyun.com/help/article/29.html)
