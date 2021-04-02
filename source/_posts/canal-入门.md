---
title: Canal入门
top: false
cover: false
toc: true
mathjax: true
categories:
  - SpringCloud - Canal
tags:
  - Canal
date: 2021-04-02 16:09:45
password:
summary:
---

# Canal入门

## 一、环境搭建

### 1 Docker

#### 1 Mysql

```shell
# 创建数据，配置目录
mkdir -p /docker/mysql/{data,conf.d}
 
# 创建配置文件
vim /docker/mysql/conf.d/my.cnf
 
[mysqld]
log_timestamps=SYSTEM
default-time-zone='+8:00'
log-bin=mysql-bin  # 开启binlog
server-id=3306  # 配置MySQL replaction需要定义，不要和Canal的slaveId重复
binlog_format=row  # 选择ROW模式
 
 
# 启动脚本
vim /docker/mysql/start.sh
 
docker run --name mysql \
-p 3306:3306 \
-v /docker/mysql/conf.d:/etc/mysql/conf.d \
-v /docker/mysql/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
-d mysql:5.7
```

1. 创建用于同步的账号

```mysql
grant SELECT, REPLICATION SLAVE, REPLICATION CLIENT on *.* to 'canal'@'%' identified by "canal";
 
flush privileges;
```
2. 测试是否生效

```shell
show variables like 'log_bin';
show variables like 'binlog_format';
show master status;
```



#### 2 Canal

1. 新建启动脚本

   > vim /docker/canal/start.sh

```shell
docker run --name canal \
-e canal.instance.master.address=42.193.118.181:3306 \
-e canal.instance.dbUsername=canal \
-e canal.instance.dbPassword=canal \
-p 11111:11111 \
-d canal/canal-server
```

2. 查看日志

```shell
docker logs --tail="100" canal 
```

