---
title: MySQL主从复制
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

#  主从复制简介

## 1 高可用架构方案

```undefined
负载均衡:有一定的高可用性 
LVS  Nginx
主备系统:有高可用性,但是需要切换,是单活的架构
KA ,   MHA, MMM
真正高可用(多活系统): 
NDB Cluster  Oracle RAC  Sysbase cluster   , InnoDB Cluster（MGR）,PXC , MGC
```

## 2 主从复制简介

```css
1.1. 基于二进制日志复制的
1.2. 主库的修改操作会记录二进制日志
1.3. 从库会请求新的二进制日志并回放,最终达到主从数据同步
1.4. 主从复制核心功能:
辅助备份,处理物理损坏                   
扩展新型的架构:高可用,高性能,分布式架构等
```

## 3 主从复制原理

主库需要二进制日志（binlog）

从库需要中级日志relaylog，master.info（主库信息）relaylog.info(中级日志)

### 3.1 线程

主库

* Binlog_Dump_Thread

从库

* Slave_IO_Thread
* SAVLE_SQL_Thread

### 3.2 原理

![image-20220321103103403](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202203211031551.png)

![img](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202203202215808)

![img](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202203202216114)

```cpp
1.change master to 时，ip pot user password binlog position写入到master.info进行记录
2. start slave 时，从库会启动IO线程和SQL线程
3.IO_T，读取master.info信息，获取主库信息连接主库
4. 主库会生成一个准备binlog DUMP线程，来响应从库
5. IO_T根据master.info记录的binlog文件名和position号，请求主库DUMP最新日志
6. DUMP线程检查主库的binlog日志，如果有新的，TP(传送)给从从库的IO_T
7. IO_T将收到的日志存储到了TCP/IP 缓存，立即返回ACK给主库 ，主库工作完成
8.IO_T将缓存中的数据，存储到relay-log日志文件,更新master.info文件binlog 文件名和postion，IO_T工作完成
9.SQL_T读取relay-log.info文件，获取到上次执行到的relay-log的位置，作为起点，回放relay-log
10.SQL_T回放完成之后，会更新relay-log.info文件。
11. relay-log会有自动清理的功能。
细节：
1.主库一旦有新的日志生成，会发送“信号”给binlog dump ，IO线程再请求
```

## 4 搭建前提

```fcss
## 2.1 两台以上mysql实例 ,server_id,server_uuid不同
## 2.2 主库开启二进制日志
## 2.3 专用的复制用户
## 2.4 保证主从开启之前的某个时间点,从库数据是和主库一致(补课)
## 2.5 告知从库,复制user,passwd,IP port,以及复制起点(change master to)
## 2.6 线程(三个):Dump thread  IO thread  SQL thread 开启(start slave)
```

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
# [必须]启用二进制日志
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
show master status \G
show slave hosts \G
```

```bash
docker-compose exec mysql-slave bash
change master to master_host='mysql-master',master_port=3306,master_user='slave',master_password='123456',master_log_file='mysql-bin.000006',master_log_pos=438; 
show slave status \G
```

# 主从故障监控\分析\处理 *****  

## 1 线程相关监控

### 1.1 主库:



```css
show full processlist;
每个从库都会有一行dump相关的信息
HOSTS: 
db01:47176
State:
Master has sent all binlog to slave; waiting for more updates
如果现实非以上信息,说明主从之间的关系出现了问题    
```

### 1.2 从库:



```css
db01 [(none)]>show slave status \G
*************************** 1. row ***************************
```

##  2 主库相关信息监控



```css
Master_Host: 10.0.0.51
Master_User: repl
Master_Port: 3307
Master_Log_File: mysql-bin.000005
Read_Master_Log_Pos: 444
```

## 3 从库中继日志的应用状态



```css
Relay_Log_File: db01-relay-bin.000002
Relay_Log_Pos: 485
```

## 4 从库复制线程有关的状态



```undefined
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
Last_IO_Errno: 0
Last_IO_Error: 
Last_SQL_Errno: 0
Last_SQL_Error: 
```

## 5 过滤复制有关的状态



```undefined
Replicate_Do_DB: 
Replicate_Ignore_DB: 
Replicate_Do_Table: 
Replicate_Ignore_Table: 
Replicate_Wild_Do_Table: 
Replicate_Wild_Ignore_Table: 
```

## 6 主从延时相关状态(非人为)



```undefined
Seconds_Behind_Master: 0
```

## 7 延时从库有关的状态(人为)



```cpp
SQL_Delay: 0
SQL_Remaining_Delay: NULL
```

## 8 GTID 复制有关的状态



```undefined
Retrieved_Gtid_Set: 
Executed_Gtid_Set: 
Auto_Position: 0
```

## 9 主从复制故障分析

### 9.1 IO

#### 9.1.1 连接主库

```csharp
(1) 用户 密码  IP  port
Last_IO_Error: error reconnecting to master 'repl@10.0.0.51:3307' - retry-time: 10  retries: 7
[root@db01 ~]# mysql -urepl  -p123333  -h 10.0.0.51 -P 3307
ERROR 1045 (28000): Access denied for user 'repl'@'db01' (using password: YES)

原因:
密码错误 
用户错误 
skip_name_resolve
地址错误
端口
```

![img](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202203202235582)

![img](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202203202235551)

 **处理方法**

```shell
stop  slave  
# 重置信息
reset slave all 
# 重新搭建主从关系
change master to 
start slave
```

#### 9.1.2  请求二进制日志

```csharp
主库缺失日志
从库方面,二进制日志位置点不对
Last_IO_Error: Got fatal error 1236 from master when reading data from binary log: 'could not find next log; the first event 'mysql-bin.000001' at 154, the last event read from '/data/3307/data/mysql-bin.000002' at 154, the last byte read from '/data/3307/data/mysql-bin.000002' at 154.'
```

![img](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202203210943801)

```undefined
注意: 在主从复制环境中,严令禁止主库中reset master; 可以选择expire 进行定期清理主库二进制日志
解决方案:
重新构建主从
```

### 9.2 主库连接数上线,或者是主库太繁忙

```dart
show slave  staus \G 
Last_IO_Errno: 1040
Last_IO_Error: error reconnecting to master 'repl@10.0.0.51:3307' - retry-time: 10  retries: 7
处理思路:
拿复制用户,手工连接一下

[root@db01 ~]# mysql -urepl -p123 -h 10.0.0.51 -P 3307 
mysql: [Warning] Using a password on the command line interface can be insecure.
ERROR 1040 (HY000): Too many connections
处理方法:
db01 [(none)]>set global max_connections=300;

(3) 防火墙,网络不通
```

### 9.3 SQL 线程故障

```cpp
(1)读写relay-log.info 
(2)relay-log损坏,断节,找不到
(3)接收到的SQL无法执行
```

**导致SQL线程故障原因分析：**

```rust
1. 版本差异，参数设定不同，比如：数据类型的差异，SQL_MODE影响
2.要创建的数据库对象,已经存在
3.要删除或修改的对象不存在  
4.DML语句不符合表定义及约束时.  
归根揭底的原因都是由于从库发生了写入操作.
Last_SQL_Error: Error 'Can't create database 'db'; database exists' on query. Default database: 'db'. Query: 'create database db'
```

**处理方法(以从库为核心的处理方案)：**

```csharp
方法一：
stop slave; 
set global sql_slave_skip_counter = 1;
#将同步指针向下移动一个，如果多次不同步，可以重复操作。
start slave;
方法二：
/etc/my.cnf
slave-skip-errors = 1032,1062,1007
常见错误代码:
1007:对象已存在
1032:无法执行DML
1062:主键冲突,或约束冲突

但是，以上操作有时是有风险的，最安全的做法就是重新构建主从。把握一个原则,一切以主库为主.
```

**一劳永逸的方法:**

```dart
(1) 可以设置从库只读.
db01 [(none)]>show variables like '%read_only%';
注意：
只会影响到普通用户，对管理员用户无效。
(2)加中间件
读写分离。
```

## 10 主从延时

在网络正常的时候，日志从主库传给从库所需的时间是很短的，即T 2 - T 1 的值是非常小的。即，网络正常情况下，主备延迟的主要来源是备库接收完binlog和执行完这个事务之间的时间差。

> **主备延迟最直接的表现是，从库消费中继日志（relay log）的速度，比主库生产binlog的速度要慢。*

**外在因素**

```undefined
网络 
主从硬件差异较大
版本差异
参数因素
```

**主库**

```css
(1) 二进制日志写入不及时
[rep]>select @@sync_binlog;
(2) CR的主从复制中,binlog_dump线程,事件为单元,串行传送二进制日志(5.6 5.5)

1. 主库并发事务量大,主库可以并行,传送时是串行
2. 主库发生了大事务,由于是串行传送,会产生阻塞后续的事务.
4  从库太多，线程传送慢
解决方案:
1. 5.6 开始,开启GTID,实现了GC(group commit)机制,可以并行传输日志给从库IO
2. 5.7 开始,不开启GTID,会自动维护匿名的GTID,也能实现GC,我们建议还是认为开启GTID
3. 大事务拆成多个小事务,可以有效的减少主从延时.
4 一般来说一主三从
```

**大事务案例**

**举例 1 ：** 一次性用delete语句删除太多数据

结论：后续再删除数据的时候，要控制每个事务删除的数据量，分成多次删除。

**举例 2 ：** 一次性用insert...select插入太多数据

**举例: 3 ：** 大表DDL

比如在主库对一张500W的表添加一个字段耗费了 10 分钟，那么从节点上也会耗费 10 分钟。

**从库**

```css
SQL线程导致的主从延时
在传动的主从复制情况下: 从库默认情况下只有一个SQL,只能串行回放事务SQL
1. 主库如果并发事务量较大,从库只能串行回放
2. 主库发生了大事务,会阻塞后续的所有的事务的运行

解决方案:
1. 5.6 版本开启GTID之后,加入了SQL多线程的特性,但是只能针对不同库(database)下的事务进行并发回放.
2. 5.7 版本开始GTID之后,在SQL方面,提供了基于逻辑时钟(logical_clock),binlog加入了seq_no机制,实现了给予事务级别的并发回访
真正实现了基于事务级别的并发回放,这种技术我们把它称之为MTS(enhanced multi-threaded slave).
3. 大事务拆成多个小事务,可以有效的减少主从延时.
[https://dev.mysql.com/worklog/task/?id=6314]
```

# 延时从库

## 1.1介绍

> 是我们认为配置的一种特殊从库.人为配置从库和主库延时N小时.

## 1.2 为什么要有延时从

```undefined
数据库故障?
物理损坏
主从复制非常擅长解决物理损坏.
逻辑损坏
普通主从复制没办法解决逻辑损坏
```

## 1.3 配置延时从库

```mysql
# 如果24小时有人值班就会很快，否则就会很慢
SQL线程延时:数据已经写入relaylog中了,SQL线程"慢点"运行
一般企业建议3-6小时,具体看公司运维人员对于故障的反应时间

mysql>stop slave;
# 单位秒
mysql>CHANGE MASTER TO MASTER_DELAY = 300;
mysql>start slave;

mysql> show slave status \G
SQL_Delay: 300
SQL_Remaining_Delay: NULL
```

## 1.4 延时从库应用

### 1.4.1 故障恢复思路

```css
1主1从,从库延时5分钟,主库误删除1个库
1. 5分钟之内 侦测到误删除操作
2. 停从库SQL线程，挂起维护页
3. 截取relaylog
起点 :停止SQL线程时,relay最后应用位置
终点:误删除之前的position(GTID)
4. 恢复截取的日志到从库
5. 从库身份解除,替代主库工作 
```

### 1.4.2 故障模拟及恢复

```css
1.主库数据操作
db01 [(none)]>create database relay charset utf8;
db01 [(none)]>use relay
db01 [relay]>create table t1 (id int);
db01 [relay]>insert into t1 values(1);
db01 [relay]>drop database relay;
```

```undefined
2. 发现故障，立即停止从库SQL线程
stop slave sql_thread; 只停止sql线程

```

```kotlin
3. 找relaylog的截取起点和终点
起点:
Relay_Log_File: db01-relay-bin.000002
Relay_Log_Pos: 482
终点:
show relaylog events in 'db01-relay-bin.000002'
| db01-relay-bin.000002 | 1046 | Xid            |         7 |        2489 | COMMIT /* xid=144 */                  |
| db01-relay-bin.000002 | 1077 | Anonymous_Gtid |         7 |        2554 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'  |
mysqlbinlog --start-position=482 --stop-position=1077  /data/3308/data/db01-relay-bin.000002>/tmp/relay.sql
```

![relaylog图](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202203211142614.png)

* pos为relaylog起点
* end_log_pos 为主库binlog起点

**如何截取relaylog？**

> 首先找到起点，其次在relog找到终点

![image-20220321114514299](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202203211145426.png)

4. 从库恢复relaylog

```bash
source /tmp/relay.sql
```

5. 从库身份解除

```css
db01 [relay]>stop slave;
db01 [relay]>reset slave all
```

# 如何解决一致性问题

## 异步复制

![异步复制](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202203211337137.png)



## 半同步复制

```css
解决主从数据一致性问题
5.5 版本出现
5.6 增强
5.7 比较完善
5.7.17 MGR复制（组复制）

```



![半同步复制](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202203211311709.png)

**工作过程**

```dart
1. 主库执行新的事务,commit时,更新 show master  status\G ,触发一个信号给
2. binlog dump 接收到主库的 show master status\G信息,通知从库日志更新了
3. 从库IO线程请求新的二进制日志事件
4. 主库会通过dump线程传送新的日志事件,给从库IO线程
5. 从库IO线程接收到binlog日志,当日志写入到磁盘上的relaylog文件时,给主库ACK_receiver线程
6. ACK_receiver线程触发一个事件,告诉主库commit可以成功了
7. 如果ACK达到了我们预设值的超时时间,半同步复制会切换为原始的异步复制.
```

## 组复制

异步复制和半同步复制都无法最终保证数据的一致性问题，半同步复制是通过判断从库响应的个数来决定是否返回给客户端，虽然数据一致性相比于异步复制有提升，但仍然无法满足对数据一致性要求高的场景，比如金融领域。MGR 很好地弥补了这两种复制模式的不足。

组复制技术，简称 MGR（MySQL Group Replication）。是 MySQL 在 5.7.17 版本中推出的一种新的数据复制技术，这种复制技术是基于 Paxos 协议的状态机复制。

![image-20220321133928107](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202203211339207.png)

**MGR 是如何工作的**

首先我们将多个节点共同组成一个复制组，在执行读写（RW）事务的时候，需要通过一致性协议层（Consensus 层）的同意，也就是读写事务想要进行提交，必须要经过组里“大多数人”（对应 Node 节
点）的同意，大多数指的是同意的节点数量需要大于 （N/2+1），这样才可以进行提交，而不是原发起方一个说了算。而针对只读（RO）事务则不需要经过组内同意，直接 COMMIT 即可。

在一个复制组内有多个节点组成，它们各自维护了自己的数据副本，并且在一致性协议层实现了原子消息和全局有序消息，从而保证组内数据的一致性。

MGR 将 MySQL 带入了数据强一致性的时代，是一个划时代的创新，其中一个重要的原因就是MGR 是基于 Paxos 协议的。Paxos 算法是由 2013 年的图灵奖获得者 Leslie Lamport 于 1990 年提出的，有关这个算法的决策机制可以搜一下。事实上，Paxos 算法提出来之后就作为分布式一致性算法被广泛应用，比如Apache 的 ZooKeeper 也是基于 Paxos 实现的。

# 过滤复制

## 说明

主库：

```dart
show master status;
Binlog_Do_DB
Binlog_Ignore_DB 
```

从库：

```dart
show slave status\G
Replicate_Do_DB: 
Replicate_Ignore_DB: 
```

**3.2 实现过程**

```csharp
mysqldump -S /data/3307/mysql.sock -A --master-data=2 --single-transaction  -R --triggers >/backup/full.sql

vim  /backup/full.sql
-- CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000002', MASTER_LOG_POS=154;

[root@db01 ~]# mysql -S /data/3309/mysql.sock 
source /backup/full.sql

CHANGE MASTER TO
MASTER_HOST='10.0.0.51',
MASTER_USER='repl',
MASTER_PASSWORD='123',
MASTER_PORT=3307,
MASTER_LOG_FILE='mysql-bin.000002',
MASTER_LOG_POS=154,
MASTER_CONNECT_RETRY=10;
start  slave;
[root@db01 ~]# vim /data/3309/my.cnf 
replicate_do_db=ppt
replicate_do_db=word
[root@db01 ~]# systemctl restart mysqld3309

主库：
Master [(none)]>create database word;
Query OK, 1 row affected (0.00 sec)
Master [(none)]>create database ppt;
Query OK, 1 row affected (0.00 sec)
Master [(none)]>create database excel;
Query OK, 1 row affected (0.01 sec)
```

# GTID复制

## GTID介绍

```css
GTID(Global Transaction ID)是对于一个已提交事务的唯一编号，并且是一个全局(主从复制)唯一的编号。
它的官方定义如下：
GTID = source_id ：transaction_id
7E11FA47-31CA-19E1-9E56-C43AA21293967:29
什么是sever_uuid，和Server-id 区别？
核心特性: 全局唯一,具备幂等性
```

## GTID核心参数

```bash
gtid-mode=on
enforce-gtid-consistency=true
log-slave-updates=1

gtid-mode=on                        --启用gtid类型，否则就是普通的复制架构
enforce-gtid-consistency=true               --强制GTID的一致性
#级联复制也要加
log-slave-updates=1                 --slave更新是否记入日志
```

## GTID复制配置过程：

```kotlin
pkill mysqld
 \rm -rf /data/mysql/data/*
 \rm -rf /data/binlog/*
```

```kotlin
主库db01：
cat > /etc/my.cnf <<EOF
[mysqld]
basedir=/data/mysql/
datadir=/data/mysql/data
socket=/tmp/mysql.sock
server_id=51
port=3306
secure-file-priv=/tmp
autocommit=0
log_bin=/data/binlog/mysql-bin
binlog_format=row
gtid-mode=on
enforce-gtid-consistency=true
log-slave-updates=1
[mysql]
prompt=db01 [\\d]>
EOF

slave1(db02)：
cat > /etc/my.cnf <<EOF
[mysqld]
basedir=/data/mysql
datadir=/data/mysql/data
socket=/tmp/mysql.sock
server_id=52
port=3306
secure-file-priv=/tmp
autocommit=0
log_bin=/data/binlog/mysql-bin
binlog_format=row
gtid-mode=on
enforce-gtid-consistency=true
log-slave-updates=1
[mysql]
prompt=db02 [\\d]>
EOF

slave2(db03)：
cat > /etc/my.cnf <<EOF
[mysqld]
basedir=/data/mysql
datadir=/data/mysql/data
socket=/tmp/mysql.sock
server_id=53
port=3306
secure-file-priv=/tmp
autocommit=0
log_bin=/data/binlog/mysql-bin
binlog_format=row
gtid-mode=on
enforce-gtid-consistency=true
log-slave-updates=1
[mysql]
prompt=db03 [\\d]>
EOF
```

###  初始化数据

```bash
mysqld --initialize-insecure --user=mysql --basedir=/data/mysql  --datadir=/data/mysql/data 
```

### 启动数据库

```bash
/etc/init.d/mysqld start
```

### 构建主从：

```csharp
master:51
slave:52,53

51:
grant replication slave  on *.* to repl@'10.0.0.%' identified by '123';

52\53:
change master to 
master_host='10.0.0.51',
master_user='repl',
master_password='123' ,
MASTER_AUTO_POSITION=1;

start slave;
```

##  GTID 复制和普通复制的区别



```bash
# 普通复制
CHANGE MASTER TO
MASTER_HOST='10.0.0.51',
MASTER_USER='repl',
MASTER_PASSWORD='123',
MASTER_PORT=3307,
MASTER_LOG_FILE='mysql-bin.000001',
MASTER_LOG_POS=444,
MASTER_CONNECT_RETRY=10;

# GTIO复制
change master to 
master_host='10.0.0.51',
master_user='repl',
master_password='123' ,
MASTER_AUTO_POSITION=1;
start slave;
# MASTER_AUTO_POSITION=1; 参数表示会读relaylog中读取relaylog最后一行的信息，去主库请求，如果第一次搭建，relaylog是空的。就会从第一个开始搭建，如果这个库已经有了一万个GTID了，这种情况还需要先应用主库的备份在搭建主从，备份时候有个包括GTID信息的参数

（0）在主从复制环境中，主库发生过的事务，在全局都是由唯一GTID记录的，更方便Failover
（1）额外功能参数（3个）
（2）change master to 的时候不再需要binlog 文件名和position号,MASTER_AUTO_POSITION=1;
（3）在复制过程中，从库不再依赖master.info文件，而是直接读取最后一个relaylog的 GTID号
（4） mysqldump备份时，默认会将备份中包含的事务操作，以以下方式
    SET @@GLOBAL.GTID_PURGED='8c49d7ec-7e78-11e8-9638-000c29ca725d:1';
    告诉从库，我的备份中已经有以上事务，你就不用运行了，直接从下一个GTID开始请求binlog就行。
```

## GTID 从库误写入操作处理

> GTID不允许断点

```bash
查看监控信息:
Last_SQL_Error: Error 'Can't create database 'oldboy'; database exists' on query. Default database: 'oldboy'. Query: 'create database oldboy'
# 从主库拿过来的GTID
Retrieved_Gtid_Set: 71bfa52e-4aae-11e9-ab8c-000c293b577e:1-3

# 已经运行的GTID
Executed_Gtid_Set:  71bfa52e-4aae-11e9-ab8c-000c293b577e:1-2,
7ca4a2b7-4aae-11e9-859d-000c298720f6:1

注入空事物的方法：

stop slave;
set gtid_next='99279e1e-61b7-11e9-a9fc-000c2928f5dd:3';
begin;commit;
set gtid_next='AUTOMATIC';
    
这里的xxxxx:N 也就是你的slave sql thread报错的GTID，或者说是你想要跳过的GTID。
最好的解决方案：重新构建主从环境
```

# 文章参考

[oldguo](https://www.jianshu.com/p/6ed2cc292077)

[使用Docker-compose 搭建 MySQL 主从 集群v](https://caiqing123.github.io/post/docker-mysql/)

[[MySQL] 用Docker搭建MySQL主从多节点集群](https://piaohua.github.io/post/mysql/20210523-mysql-master-slaves/)

