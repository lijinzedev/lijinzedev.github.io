---
title: RocketMQ-docker-集群笔记
top: false
cover: false
toc: true
mathjax: true
categories:
  - 中间件
  - mq
  - rocketmq 
tags:
  - 中间件
date: 2022-07-22 21:33:58
password:
summary:
---

# M1使用docker部署rocketmq单机版

https://www.jianshu.com/p/6ad529a16677

## 1. 编译rocketmq镜像

- 先down下代码 ：git clone [https://github.com/apache/rocketmq-docker.git](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fapache%2Frocketmq-docker.git)
- 按照readme中的步骤执行命令

```bash
cd image-build
sh build-image.sh RMQ-VERSION BASE-IMAGE
(我执行的:sh build-image.sh 4.8.0 alpine )
```

* 上一步成功后运行 docker images

![img](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202207222137492)

## 2.编译rocketmq-console-ng镜像

* 先down代码： git clone git@github.com:apache/rocketmq-dashboard.git
* 执行 `mvn clean package -Dmaven.test.skip=true` 会在./target目录下生成一个jar包
* 将jar包copy到./src/main/docker，然后 进入该目录执行 `docker build -t rocketmq-console-ng:2.0 .`

```dockerfile
FROM openjdk:8-jre
VOLUME /tmp
ADD *.jar app.jar
RUN sh -c 'touch /app.jar'
ENV JAVA_OPTS=""
ENTRYPOINT ["sh","-c","java $JAVA_OPTS -jar /app.jar"]
```

![image-20220722214058823](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202207222140882.png)

## 3.broker.conf配置

由于咱们是docker构建服务，我在使用第三方rocketmq-client包创建生产者时总是使用docker分配的内网ip进行连接，因连接不通，所以我们需要修改broker配置，使其用本地ip进行连接。具体配置如下：

```css
brokerClusterName = rocketmq-cluster 
brokerName = broker-a
brokeId = 0
deleteWhen = 04
fileReservedTime = 48
brokerRole = ASYNC_MASTER
flushDiskType = ASYNC_FLUSH
brokerIP1 = 192.168.173.3
```

## 4. docker-compose配置

```yaml
version: '3'
services:
  namesrv:
    image: apacherocketmq/rocketmq:4.8.0-alpine
    container_name: rmqnamesrv
    ports:
      - 9876:9876
    command: sh mqnamesrv
  broker:
    image: apacherocketmq/rocketmq:4.8.0-alpine //这儿是自己编译的镜像
    container_name: rmqbroker
    ports:
      - 10909:10909
      - 10911:10911
      - 10912:10912
    volumes:
    //这儿将本地的broker.conf配置文件挂在到容器中
      - '本机broker.conf目录'/broker.conf:/home/rocketmq/rocketmq-4.8.0/conf/broker.conf
    command: sh mqbroker -n namesrv:9876 -c /home/rocketmq/rocketmq-4.8.0/conf/broker.conf
    depends_on:
      - namesrv
  mqconsole:
    image: candice0630/rocketmq-console-ng:2.0 //这儿是自己编译的ng镜像
    container_name: rmqconsole
    ports:
      - 19876:8080
    environment:
      JAVA_OPTS: -Drocketmq.config.namesrvAddr=namesrv:9876 -Drocketmq.config.isVIPChannel=false
    depends_on:
      - namesrv

```

## 5. 运行

使用docker-compose执行第三部配置好的yml文件
eg：`docker-compose -f rocketmq.yml up -d`

