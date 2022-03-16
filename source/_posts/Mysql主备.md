---
title: MySQL主备
top: false
cover: false
toc: true
mathjax: true
categories:
  - mysql
tags:
  - mysql
date: 2022-03-15 16:00:33
password:
summary:
---

# MySQL主备搭建

# 环境搭建

* M1 docker
* mysql5.7 



## 1 创建目录结构

```fallback
- docker-compose.yaml
- mysql/master/config
- mysql/master/data
- mysql/slave/config
- mysql/slave/data
```

```shell
mkdir -p  master/{data,config}
mkdir -p  slave/{data,config}
```

```mysql

```



## 2 创建docker-compose文件

```yaml
version: '3'
services:
  mysql-master:
    image: mysql/mysql-server:5.7
    container_name: mysql-master
    volumes:
      - ./master/config/my.cnf:/etc/my.cnf
      - ./master/data:/var/lib/mysql
    ports:
      - "13309:3306"
    environment:
      MYSQL_USER: admin
      MYSQL_PASSWORD: 123456
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_ROOT_HOST: '%'
    links:
      - mysql-slave  
  mysql-slave:
    image: mysql/mysql-server:5.7
    container_name: mysql-slave
    volumes:
      - ./slave/config/my.cnf:/etc/my.cnf
      - ./slave/data:/var/lib/mysql
    ports:
      - "13310:3306"
    environment:
      MYSQL_USER: admin
      MYSQL_PASSWORD: 123456
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_ROOT_HOST: '%'
```

## 3 主从配置文件

### 3.1 主库

```
[client]
port                    = 3306
default-character-set   = utf8mb4

[mysqld]
user                    = mysql
port                    = 3306
sql_mode                = ""
server_id               = 1 ## 全局id 这里用来区分
log_bin                 = mysql-bin
# 需要同步的数据库，如果不配置则同步全部数据库
#binlog-do-db=test_db
expire_logs_days        = 10  ## 存放天数 超过天数丢弃
binlog_format           = ROW ## 日志格式

default-storage-engine  = InnoDB
default-authentication-plugin   = mysql_native_password
character-set-server    = utf8mb4
collation-server        = utf8mb4_unicode_ci
init_connect            = 'SET NAMES utf8mb4'


slow_query_log
long_query_time         = 3
slow-query-log-file     = /var/lib/mysql/mysql.slow.log
log-error               = /var/lib/mysql/mysql.error.log

default-time-zone       = '+8:00'

[mysql]
default-character-set   = utf8mb4
```

### 3.2 从库

```
[client]
port                    = 3306
default-character-set   = utf8mb4

[mysqld]
user                    = mysql
port                    = 3306
sql_mode                = ""
server_id               = 2 ## 全局id 不能和其他库的一样
log_bin                 = mysql-bin
binlog_format           = ROW 
log_slave_updates        = ON ## 默认没有开启, 是从库自己也更新binlog
read_only                = ON ## 从库只读模式
# super_read_only            = ON ## 通用只读但是是super也一样


default-storage-engine  = InnoDB
default-authentication-plugin   = mysql_native_password
character-set-server    = utf8mb4
collation-server        = utf8mb4_unicode_ci
init_connect            = 'SET NAMES utf8mb4'

slow_query_log
long_query_time         = 3
slow-query-log-file     = /var/lib/mysql/mysql.slow.log
log-error               = /var/lib/mysql/mysql.error.log

default-time-zone       = '+8:00'

[mysql]
default-character-set   = utf8mb4
```

## 4 相关操作

```bash
# 首先进入主库容器
docker-compose exec mysql-master bash
GRANT REPLICATION SLAVE ON *.* to 'slave'@'%' identified by '123456'; # 这里mysql-slave 换成mysql-slave的IP
```

```bash
docker-compose exec mysql-slave bash
change master to master_host='mysql-master',master_port=3306,master_user='slave',master_password='123456',master_log_file='mysql-bin.000006',master_log_pos=438; 

```

# 文章参考

[使用Docker-compose 搭建 MySQL 主从 集群v'vvvv](https://caiqing123.github.io/post/docker-mysql/)

[[MySQL] 用Docker搭建MySQL主从多节点集群](https://piaohua.github.io/post/mysql/20210523-mysql-master-slaves/)
