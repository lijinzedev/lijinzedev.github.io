---
title: Docker 高级
top: false
cover: false
toc: true
mathjax: true
categories:
  - docker
tags:
  - docker
date: 2021-04-13 15:54:09
password:
summary:
---

# 一、Docker Compose

Using Compose is basically a three-step process:

1. Define your app’s environment with a `Dockerfile` so it can be reproduced anywhere.
2. Define the services that make up your app in `docker-compose.yml` so they can be run together in an isolated environment.
3. Run `docker compose up` and the [Docker compose command](https://docs.docker.com/compose/cli-command/) starts and runs your entire app. You can alternatively run `docker-compose up` using the docker-compose binary.

```yaml
version: "3.9"  # optional since v1.27.0
services:
  web:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - .:/code
      - logvolume01:/var/log
    links:
      - redis
  redis:
    image: redis
volumes:
  logvolume01: {}
```

Compose:

> 对外服务的最小单元

* 服务Services: 容器、应用,(web、redis、mysql)

* 项目project：一组关联的容器





# 二、环境搭建

## 2.1 安装

```shell
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

## 2.2 授权

```shell
sudo chmod +x /usr/local/bin/docker-compose
```

## 2.3 [官方案例](https://docs.docker.com/compose/gettingstarted/)

# 三、Compose的编写规则

[官方文档](https://docs.docker.com/compose/compose-file/compose-file-v3/#build)

```yaml
# 版本
version: "3.9"
# 服务
services:
	服务1:
		服务1配置:
	服务2:
		服务2配置:
# 其他配置
volumes:
networks:
configs:
```

