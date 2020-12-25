---
title: 'IDEA集成Docker与远程Debug'
top: false
cover: false
toc: true
mathjax: true
categories:
  - docker
tags:
  - docker
date: 2020-12-23 21:27:30
password:
summary:
---

# 1 使用Dockerfile运行

## 环境

* **IDEA 2020.1** 
* **Docker服务器 Centos 7**
* **Docker版本 20.10.1**
* **JDK 8**

## 环境准备

开启Docker远程访问

```bash
# 1 编辑文件
vi /usr/lib/systemd/system/docker.service
# 2 找到 ExecStart 修改为
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock
# 3 使用telnet <ip> <port> 进行测试
```

![配置成功图](image-20201225235204060.png)



## 第一步

![添加Docker](image-20201225233449444.png)

## 第二步

![选择DockerFile](image-20201225233557768.png)

## 第三步

![进行配置](image-20201225233804019.png)

## 第四步

配置命令

```bash
clean package -U -DskipTest
```

![添加](image-20201225233936031.png)

## 第五步

> 在根目录创建并写好Dockerfile

```dockerfile

### 基础镜像，使用alpine操作系统，openjkd使用8u201
FROM openjdk:8u201-jdk-alpine3.9

#作者
MAINTAINER LJZ

#系统编码
ENV LANG=C.UTF-8 LC_ALL=C.UTF-8

#声明一个挂载点，容器内此路径会对应宿主机的某个文件夹
VOLUME /tmp

#应用构建成功后的jar文件被复制到镜像内，名字也改成了app.jar
ADD target/demo-0.0.1-SNAPSHOT.jar app.jar

#暴露8080端口
EXPOSE 8080


```

# 2 远程调试容器程序

> 在上一步的基础上进行远程调试

## 第一步

![添加远程调试](image-20201225234307920.png)

## 第二步

![配置远程访问](image-20201225234612668.png)

## 第三步

![修改配置信息](image-20201225234743281.png)

## 第四步

目前Docker文件配置

```dockerfile
# Docker image for springboot application
# VERSION 0.0.1
# Author: bolingcavalry

### 基础镜像，使用alpine操作系统，openjkd使用8u201
FROM openjdk:8u201-jdk-alpine3.9

#作者
MAINTAINER LJZ

#系统编码
ENV LANG=C.UTF-8 LC_ALL=C.UTF-8

#声明一个挂载点，容器内此路径会对应宿主机的某个文件夹
VOLUME /tmp

#应用构建成功后的jar文件被复制到镜像内，名字也改成了app.jar
ADD target/demo-0.0.1-SNAPSHOT.jar app.jar


#暴露8080端口
EXPOSE 8080


#启动容器时的进程
ENTRYPOINT ["java","-jar","-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=8080","/app.jar"]

```

