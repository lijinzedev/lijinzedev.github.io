---
title: kafka 集群环境搭建
top: false
cover: false
toc: true
mathjax: true
categories:
  - 环境搭建
  - kafka
tags:
  - 环境搭建
date: 2022-06-13 17:14:17
password:
summary:
---

## 一、原生Docker命令

1. 删除所有dangling数据卷（即无用的Volume，僵尸文件）

   ```bash
   docker volume rm $(docker volume ls -qf dangling=true)
   ```

2.  删除所有dangling镜像（即无tag的镜像）

   ```bash
   docker rmi $(docker images | grep "^<none>" | awk "{print $3}"
   ```

3. 删除所有关闭的容器

   ```bash
   docker ps -a | grep Exit | cut -d ' ' -f 1 | xargs docker rm
   ```

## 二、镜像选择

环境为M1版本的mbp:

- Zookeeper采用zookeeper
- Kafka采用wurstmeister/kafka
- Kafka-Manager采用scjtqs/kafka-manager
- MySQL采用mysql/mysql-server



## 三、集群规划

| hostname          | port                         | listener |
| :---------------- | :--------------------------- | :------- |
| zook1             | 2184:2181                    |          |
| zook2             | 2185:2181                    |          |
| kafka1            | 内部9092:9092，外部9192:9192 | kafka1   |
| kafka2            | 内部9093:9093，外部9193:9193 | kafka2   |
| 本机（宿主机Mbp） |                              |          |
| kafka manager     | 9000:9000                    |          |

## 四、集群搭建

```bash
mkdir  -p ./zkcluster/{zook1,zook2}/{data,datalog}
mkdir  -p ./kafka/{kafka1,kafka2}/{wurstmeister/kafka,kafka}
```

```yml
version: "3"
services:
   zook1:
     image: zookeeper:latest
     restart: always
     hostname: zook1
     container_name: zook1 #容器名称，方便在rancher中显示有意义的名称
     ports:
       - 2183:2181 #将本容器的zookeeper默认端口号映射出去
     volumes: # 挂载数据卷
       - "./zkcluster/zook1/data:/data"
       - "./zkcluster/zook1/datalog:/datalog"
       - "./zkcluster/zook1/logs:/logs"
     environment:
       ZOO_MY_ID: 1    #即是zookeeper的节点值，也是kafka的brokerid值
       ZOO_SERVERS: server.1=zook1:2888:3888;2181 server.2=zook2:2888:3888;2181
   zook2:
      image: zookeeper:latest
      restart: always
      hostname: zook2
      container_name: zook2 #容器名称，方便在rancher中显示有意义的名称
      ports:
        - 2184:2181 #将本容器的zookeeper默认端口号映射出去
      volumes:
        - ./zkcluster/zook2/data:/data
        - ./zkcluster/zook2/datalog:/datalog
        - ./zkcluster/zook2/logs:/logs
      environment:
        ZOO_MY_ID: 2    #即是zookeeper的节点值，也是kafka的brokerid值
        ZOO_SERVERS: server.1=zook1:2888:3888;2181 server.2=zook2:2888:3888;2181
   kafka1:
    image: docker.io/wurstmeister/kafka
    restart: always
    hostname: kafka1
    container_name: kafka1
    ports:
        - 9093:9093
        - 9193:9193
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_LISTENERS: INSIDE://:9093,OUTSIDE://:9193
        #KAFKA_ADVERTISED_LISTENERS=INSIDE://<container>:9092,OUTSIDE://<host>:9094
      KAFKA_ADVERTISED_LISTENERS: INSIDE://kafka1:9093,OUTSIDE://localhost:9193
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INSIDE:PLAINTEXT,OUTSIDE:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: INSIDE
      KAFKA_ZOOKEEPER_CONNECT: zook1:2181,zook2:2181
      ALLOW_PLAINTEXT_LISTENER : 'yes'
      #JMX_PORT: 9999 #开放JMX监控端口，来监测集群数据
    volumes:
        - ./kafka/kafka1/wurstmeister/kafka:/wurstmeister/kafka
        - ./kafka/kafka1/kafka:/kafka
    depends_on:
      - zook1
      - zook2
   kafka2:
    image: docker.io/wurstmeister/kafka
    restart: always
    hostname: kafka2
    container_name: kafka2
    ports:
        - 9094:9094
        - 9194:9194
    environment:
      KAFKA_BROKER_ID: 2
      KAFKA_LISTENERS: INSIDE://:9094,OUTSIDE://:9194
        #KAFKA_ADVERTISED_LISTENERS=INSIDE://<container>:9092,OUTSIDE://<host>:9094
      KAFKA_ADVERTISED_LISTENERS: INSIDE://kafka2:9094,OUTSIDE://localhost:9194
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INSIDE:PLAINTEXT,OUTSIDE:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: INSIDE
      KAFKA_ZOOKEEPER_CONNECT: zook1:2181,zook2:2181
      ALLOW_PLAINTEXT_LISTENER : 'yes'
     # JMX_PORT: 9999 #开放JMX监控端口，来监测集群数据#
    volumes:
        - ./kafka/kafka2/wurstmeister/kafka:/wurstmeister/kafka
        - ./kafka/kafka2/kafka:/kafka
    depends_on:
      - zook1
      - zook2        
   kafka-manager:
    image: scjtqs/kafka-manager:latest
    restart: always
    hostname: kafka-manager
    container_name: kafka-manager
    ports:
        - 9000:9000
    environment:
      ZK_HOSTS: zook1:2181,zook2:2181
      KAFKA_BROKERS: kafka1:9093,kafka2:9094
      APPLICATION_SECRET: letmein
      KM_ARGS: -Djava.net.preferIPv4Stack=true
    depends_on:
      - zook1
      - zook2
      - kafka1
      - kafka2

```

```bash
# 启动
docker-compose -f  kafka-cluster.yml up
```



###  listeners 和 advertised.listeners

- `listeners`: 学名叫监听器，其实就是告诉外部连接者要通过什么协议访问指定主机名和端口开放的 `Kafka` 服务。
- `advertised.listeners`：和 `listeners` 相比多了个 `advertised`。`Advertised` 的含义表示宣称的、公布的，就是说这组监听器是 `Broker` 用于对外发布的。

比如说：

```b
   listeners: INSIDE://172.17.0.10:9092,OUTSIDE://172.17.0.10:9094
   advertised_listeners: INSIDE://172.17.0.10:9092,OUTSIDE://<公网 ip>:端口
   kafka_listener_security_protocol_map: "INSIDE:SASL_PLAINTEXT,OUTSIDE:SASL_PLAINTEXT"
   kafka_inter_broker_listener_name: "INSIDE"

```

`advertised_listeners` 监听器会注册在 `zookeeper` 中；

当我们对 `172.17.0.10:9092` 请求建立连接，`kafka` 服务器会通过 `zookeeper` 中注册的监听器，找到 `INSIDE` 监听器，然后通过 `listeners` 中找到对应的 通讯 `ip` 和 端口；

同理，当我们对 `<公网 ip>:端口` 请求建立连接，`kafka` 服务器会通过 `zookeeper` 中注册的监听器，找到 `OUTSIDE` 监听器，然后通过 `listeners` 中找到对应的 通讯 `ip` 和 端口 `172.17.0.10:9094`；

总结：`advertised_listeners` 是对外暴露的服务端口，真正建立连接用的是 `listeners`。

## 五、测试

```bash

# 随便进入一个容器
docker exec -it kafka1 /bin/bash
# 创建一个主题 qy ，3个分区，3个副本
kafka-topics.sh --bootstrap-server kafka1:9093,kafka2:9094 --create --topic qy --partitions 2 --replication-factor 2
# 往主题 qy 里面发送消息
kafka-console-producer.sh --bootstrap-server kkafka1:9093,kafka2:9094 --topic qy
# 再随便进入一个容器
docker exec -it kafka2 /bin/bash
# 查看主题 qy 的消息
kafka-console-consumer.sh --bootstrap-server kafka1:9093,kafka2:9094 --from-beginning --topic qy

```



## 六、参考

https://blog.csdn.net/sinat_36053757/article/details/123724748

## 七、非集群

```yml
version: "3"
services:
   zook1:
     image: wurstmeister/zookeeper
     restart: on-failure
     hostname: zook1
     container_name: zook1 #容器名称，方便在rancher中显示有意义的名称
     ports:
       - 2181:2181 #将本容器的zookeeper默认端口号映射出去
   kafka1:
    image: docker.io/wurstmeister/kafka
    restart: always
    hostname: kafka1
    container_name: kafka1
    ports:
        - 9092:9092
        - 9192:9192        
    environment:
      KAFKA_BROKER_ID: 0
      KAFKA_ZOOKEEPER_CONNECT: zook1:2181
      KAFKA_ADVERTISED_LISTENERS: INSIDE://kafka1:9092,OUTSIDE://localhost:9192
      KAFKA_LISTENERS: INSIDE://:9092,OUTSIDE://:9192
      KAFKA_INTER_BROKER_LISTENER_NAME: INSIDE
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INSIDE:PLAINTEXT,OUTSIDE:PLAINTEXT
      #JMX_PORT: 9999 #开放JMX监控端口，来监测集群数据
    depends_on:
      - zook1


```

