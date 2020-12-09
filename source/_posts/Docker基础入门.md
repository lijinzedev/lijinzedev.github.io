---
title: Docker基础入门
top: false
cover: false
toc: true
mathjax: true
categories:
  - docker
tags:
  - docker
date: 2020-12-08 22:24:10
password:
summary:
---

# 1 Docker基础入门

## 1 安装

### 1 安装docker
```bash
#1.卸载旧版本
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine

#2.需要的安装包
yum install -y yum-utils

#3.设置镜像的仓库
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
#默认是从国外的，不推荐
#推荐使用国内的
yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

#更新yum软件包索引
yum makecache fast

#4.安装docker相关的(docker-ce 社区版 而ee是企业版)
yum install docker-ce docker-ce-cli containerd.io

#5.启动Docker
systemctl start docker

#6.使用docker version查看是否按照成功
docker version

#7.开机自启
systemctl enable docker
```

### 2 配置阿里云镜像加速
```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://4bj04jx5.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 2 命令

### 1 帮助命令

* `docker version`
* `docker info`
* `docker --help`

### 2  镜像命令

* `docker images` 列出本地的镜像
  * `-a ` 列出本地所有的镜像（含中间映像层）
  * `-q` 只显示镜像ID
  * `--digests` 显示镜像的摘要信息
  * `--no-trunc` 显示完整的镜像信息
* `docker search <某个镜像的名字>`  
  * 



