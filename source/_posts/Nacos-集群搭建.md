---
title: Nacos 集群搭建
top: false
cover: false
toc: true
mathjax: true
categories:
  - 集群
  - Nacos
tags:
  - 集群
date: 2022-05-19 13:46:32
password:
summary:
---

# Nacos集群搭建

# 官方文档

[https://nacos.io/zh-cn/docs/quick-start-docker.html](https://nacos.io/zh-cn/docs/quick-start-docker.html)

## 环境

* M1
* Nacos 2.0.3

## 步骤

```shell
# 拉取镜像
docker pull zhusaidong/nacos-server-m1:2.0.3
```

# Docker-Compose搭建Nacos单机版本

```bash
mkdir  -p ./mysql_53306/{log,data,conf}:
mkdir -p ./nacos_8848/{standalone-logs,init.d}
touch  ./nacos_8848/init.d/custom.properties
```

`nacos-standalone.yml`

```yml
version: '3'
services:
  mysql:
    image: mysql/mysql-server:5.7
    container_name: mysql_53306
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: nacos
      MYSQL_USER: root
      MYSQL_PASSWORD: 123456
    ports:
      - 53306:3306
    volumes:
      - ./mysql_53306/log:/var/log/mysql
      - ./mysql_53306/data:/var/lib/mysql/
      - ./mysql_53306/conf/:/etc/mysql/
  nacos:
    restart: on-failure
    image: zhusaidong/nacos-server-m1:2.0.3
    container_name: nacos_8848
    depends_on:
      - mysql_53306
    environment:
      - PREFER_HOST_MODE=hostname
      - MODE=standalone
      - SPRING_DATASOURCE_PLATFORM=mysql
      - MYSQL_SERVICE_HOST=mysql_53306
      - MYSQL_SERVICE_PORT=3306
      - MYSQL_SERVICE_USER=root
      - MYSQL_SERVICE_PASSWORD=123456
      - MYSQL_SERVICE_DB_NAME=nacos
      - MYSQL_SERVICE_DB_PARAM=characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useSSL=false
    volumes:
      - ./nacos_8848/standalone-logs/:/home/nacos/logs
      - ./nacos_8848/init.d/custom.properties:/home/nacos/init.d/custom.properties
    ports:
      - "8848:8848"
      - "9848:9848"

```

**运行**

```bash
 docker-compose -f nacos-standalone.yml -p nacos-standalone  up -d
```

# Docker-Compose搭建Nacos集群版本

```bash
mkdir  -p ./nacos-cluster/mysql_53306/{log,data,conf}
mkdir  -p ./nacos-cluster/nginx/{log,html}
mkdir -p ./nacos-cluster/{cluster-logs,init.d}
touch  ./nacos-cluster/init.d/custom.properties
touch  ./nacos-cluster/nginx/nginx.conf
mkdir -p ./nacos-cluster/cluster-logs/{nacos1,nacos2,nacos3}
```



```yaml
version: "3"
services:
  nacos1:
    hostname: nacos1
    container_name: nacos1
    # 替换成m1版本的
    image: zhusaidong/nacos-server-m1:2.0.3
    volumes:
      - ./nacos-cluster/cluster-logs/nacos1:/home/nacos/logs
      - ./nacos-cluster/init.d/custom.properties:/home/nacos/init.d/custom.properties
    ports:
      - "8848:8848"
      - "9848:9848"
      - "9555:9555"
    environment: #nacos dev env
      - PREFER_HOST_MODE=hostname
      - NACOS_SERVERS=nacos1:8848 nacos2:8848 nacos3:8848
      - MYSQL_SERVICE_HOST=mysql_53306
      - MYSQL_SERVICE_DB_NAME=nacos
      - MYSQL_SERVICE_PORT=3306
      - MYSQL_SERVICE_USER=root
      - MYSQL_SERVICE_PASSWORD=123456
    restart: always
    depends_on:
      - mysql_53306
  nacos2:
    hostname: nacos2
    image: zhusaidong/nacos-server-m1:2.0.3
    container_name: nacos2
    volumes:
      - ./nacos-cluster/cluster-logs/nacos2:/home/nacos/logs
      - ./nacos-cluster/init.d/custom.properties:/home/nacos/init.d/custom.properties
    ports:
      - "8849:8848"
      - "9849:9848"
    environment: #nacos dev env
      - PREFER_HOST_MODE=hostname
      - NACOS_SERVERS=nacos1:8848 nacos2:8848 nacos3:8848
      - MYSQL_SERVICE_HOST=mysql_53306
      - MYSQL_SERVICE_DB_NAME=nacos
      - MYSQL_SERVICE_PORT=3306
      - MYSQL_SERVICE_USER=root
      - MYSQL_SERVICE_PASSWORD=123456
    restart: always
    depends_on:
      - mysql_53306
  nacos3:
    hostname: nacos3
    image: zhusaidong/nacos-server-m1:2.0.3
    container_name: nacos3
    volumes:
      - ./nacos-cluster/cluster-logs/nacos3:/home/nacos/logs
      - ./nacos-cluster/init.d/custom.properties:/home/nacos/init.d/custom.properties
    ports:
      - "8850:8848"
      - "9850:9848"
    environment: #nacos dev env
      - PREFER_HOST_MODE=hostname
      - NACOS_SERVERS=nacos1:8848 nacos2:8848 nacos3:8848
      - MYSQL_SERVICE_HOST=mysql_53306
      - MYSQL_SERVICE_DB_NAME=nacos
      - MYSQL_SERVICE_PORT=3306
      - MYSQL_SERVICE_USER=root
      - MYSQL_SERVICE_PASSWORD=123456
    restart: always
    depends_on:
      - mysql_53306
  # mysql
  mysql:
    image: mysql/mysql-server:5.7
    container_name: mysql_53306
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: nacos
      MYSQL_USER: root
      MYSQL_PASSWORD: 123456
    ports:
      - 53306:3306
    volumes:
      - ./mysql_53306/log:/var/log/mysql
      - ./mysql_53306/data:/var/lib/mysql/
      - ./mysql_53306/conf/:/etc/mysql/
  # nginx
  nginx:
    image: arm64v8/nginx
    links:
      - nacos1:nacos1
      - nacos2:nacos2
      - nacos3:nacos3
    container_name: nginx_1000
    restart: always
    depends_on:
      - nacos1
      - nacos2
      - nacos3
    ports:
      - 1000:80
    volumes:
      - ./nacos-cluster/nginx/log:/etc/nginx/logs
      - ./nacos-cluster/nginx/html:/etc/nginx/html
      - ./nacos-cluster/nginx/nginx.conf:/etc/nginx/nginx.conf

```

```nginx
#user  nobody;
worker_processes  8;
worker_rlimit_nofile 12000;
events {
    use epoll;
    worker_connections  12000;
    multi_accept    on;
}

http  {

    include     mime.types;
    default_type        application/octet-stream;


    sendfile        on;
    tcp_nopush     on;
    tcp_nodelay on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;
    upstream cluster {
        server nacos1:8848;
        server nacos2:8849;
        server nacos3:8850;
    }

     server {
        listen      80;
        server_name  localhost;
        location / {
          proxy_pass http://cluster;
      }


        access_log  logs/access.log;
        #开启索引功能
        autoindex on;
        # 关闭计算文件确切大小（单位bytes），只显示大概大小（单位kb、mb、gb）
        autoindex_exact_size off;
        autoindex_localtime on;   # 显示本机时间而非 GMT 时间
         # 解决跨域问题
        add_header Access-Control-Allow-Origin '*';
  }

}

```

```
 docker-compose -f nacos-cluster.yml -p nacos-cluster  up -d
```



https://juejin.cn/post/7081722267176009736
