---
title: docker常用命令
top: false
cover: false
toc: true
mathjax: true
categories:
  - 运维
  - docker
tags:
  - 运维
  - docker
date: 2021-04-19 14:20:21
password:
summary:
---

# Docker 常用命令

## 搜索镜像

```shell
docker search ngxin
```

## 下载镜像

```shell
docker pull java:8
```

## 删除镜像

- 指定名称删除镜像

```bash
docker rmi java:8
```

- 指定名称删除镜像（强制）

```bash
docker rmi -f java:8
```

- 删除所有没有引用的镜像

```bash
docker rmi `docker images | grep none | awk '{print $3}'`
```

- 强制删除所有镜像

```bash
docker rmi -f $(docker images)
```

## 新建并启动容器

```shell
docker run -p 80:80 --name nginx -d nginx:1.17.0
```

## 停止容器

```bash
# $ContainerName及$ContainerId可以用docker ps命令查询出来
docker stop $ContainerName(或者$ContainerId)
```

## 强制停止容器

```bash
docker kill $ContainerName(或者$ContainerId)
```

## 启动已停止的容器

```bash
docker start $ContainerName(或者$ContainerId)
```

## 进入容器

- 先查询出容器的pid：

```bash
docker inspect --format "{{.State.Pid}}" $ContainerName(或者$ContainerId)
```

- 根据容器的pid进入容器：

```bash
nsenter --target "$pid" --mount --uts --ipc --net --pid
```

## 删除容器

- 删除指定容器：

```bash
docker rm $ContainerName(或者$ContainerId)
```

- 按名称删除容器

```bash
docker rm `docker ps -a | grep test-* | awk '{print $1}'`
# 例如
docker rm `docker ps -a | grep java-* | awk '{print $1}'`
```

- 强制删除所有容器；

```bash
docker rm -f $(docker ps -a -q)
```

## 查看容器的日志

- 查看当前全部日志

```bash
docker logs $ContainerName(或者$ContainerId)
```

- 动态查看日志

```bash
docker logs $ContainerName(或者$ContainerId) -f
```

## 查看容器的IP地址

```bash
docker inspect --format '{{ .NetworkSettings.IPAddress }}' $ContainerName(或者$ContainerId)
```

## 修改容器的启动方式

```bash
docker container update --restart=always $ContainerName
# --restart=always 当 Docker 重启时，容器能自动启动。
```

## 同步宿主机时间到容器

```bash
docker cp /etc/localtime $ContainerName(或者$ContainerId):/etc/
```

## 指定容器时区

```bash
docker run -p 80:80 --name nginx \
-e TZ="Asia/Shanghai" \
-d nginx:1.17.0
```

## 在宿主机查看docker使用cpu、内存、网络、io情况

- 查看指定容器情况：

```bash
docker stats $ContainerName(或者$ContainerId)
```

- 查看所有容器情况：

```bash
docker stats -a
```

## 查看Docker磁盘使用情况

```bash
docker system df
```

## 进入Docker容器内部的bash

```bash
docker exec -it $ContainerName /bin/bash
```

## 使用root帐号进入Docker容器内部

```bash
docker exec -it --user root $ContainerName /bin/bash
```

## Docker创建外部网络

```bash
docker network create -d bridge my-bridge-network
```

## 修改Docker镜像的存放位置

- 查看Docker镜像的存放位置：

```bash
docker info | grep "Docker Root Dir"
```

- 关闭Docker服务：

```bash
systemctl stop docker
```

- 移动目录到目标路径：

```bash
mv /var/lib/docker /home/docker
```

- 建立软连接：

```bash
 ln -s /home/win10/docker-images/docker /var/lib/docker
```