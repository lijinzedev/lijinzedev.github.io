---
title: WSL2安装基本环境
top: false
cover: false
toc: true
mathjax: true
categories:
  - WSL
tags:
  - WSL
date: 2021-04-14 15:38:20
password:
summary:
---

# 换源

默认的安装源相对国内很慢，我们更换源到阿里云，登录到ubuntu到操作如下：

```shell
cp /etc/apt/sources.list /etc/apt/sources.list.bak

echo "deb http://mirrors.aliyun.com/ubuntu/ focal main restricted
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted
deb http://mirrors.aliyun.com/ubuntu/ focal universe
deb http://mirrors.aliyun.com/ubuntu/ focal-updates universe
deb http://mirrors.aliyun.com/ubuntu/ focal multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-updates multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted
deb http://mirrors.aliyun.com/ubuntu/ focal-security universe
deb http://mirrors.aliyun.com/ubuntu/ focal-security multiverse">/etc/apt/sources.list
```

# 安装JAVA环境

## 1 安装

```shell
sudo apt install openjdk-11-jdk
sudo apt install openjdk-8-jdk
```

## 2 切换版本

```shell
sudo update-alternatives --config java
java -version
```

# 安装Maven环境

```shell
 sudo apt-get install maven
```

