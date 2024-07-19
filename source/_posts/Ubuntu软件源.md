---
title: Ubuntu软件源
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
date: 2024-04-5 23:33:57
password:
summary:
---

# 一、修改 apt 源

### 查看linux系统发行版本信息

#### ubuntu 20.04

```bash
$ lsb_release -a
Distributor ID:	Ubuntu
Description:	Ubuntu 20.04.1 LTS
Release:	    20.04
Codename:	    focal
```

可以看到发行版本代号为 focal

#### ubuntu 18.04

```bash
$ lsb_release -c
Codename:	bionic
```

可以看到发行版本代号为 bionic

#### 部分ubuntu系统LTS版本代号

Ubuntu 16.04代号为：xenial

Ubuntu 17.04代号为：zesty

Ubuntu 18.04代号为：bionic

Ubuntu 19.04代号为：disco

Ubuntu 20.04代号为：focal

Ubuntu 20.04代号为：focal

Ubuntu 22.04代号为:  jammy

### 修改sources.list文件

```bash
$ sudo vim /etc/apt/sources.list
```

### 更新

```bash
$ sudo apt-get update
$ sudo apt-get upgrade
```

# ubuntu 22.04 清华大学镜像

```bash
sudo bash -c "cat << EOF > /etc/apt/sources.list && apt update 
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
EOF"
```





[ubuntu 22.04国内镜像阿里云/163源/清华大学/中科大](https://www.myfreax.com/ubuntu-22-04geng-gai-jing-xiang-ruan-jian-yuan/)

[WSL2 配置 C 语言开发环境](https://blog.huohaodong.com/blog/wsl2-development-environment-configuration)
