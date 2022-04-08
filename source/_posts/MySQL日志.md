---
title: MySQL日志
top: false
cover: false
toc: true
mathjax: true
categories:
  - mysql
tags:
  - mysql
date: 2022-03-17 16:12:04
password:
summary:
---


**千万不要小看日志**。很多看似奇怪的问题，答案往往就藏在日志里。很多情况下，只有通过查看日志才 能发现问题的原因，真正解决问题。所以，一定要学会查看日志，养成检查日志的习惯，对提升你的数 据库应用开发能力至关重要。

MySQL8.0 官网日志地址：https://dev.mysql.com/doc/refman/8.0/en/server-logs.html)

# MySQL支持的日志

## 1 日志类型

MySQL有不同类型的日志文件，用来存储不同类型的日志，分为 二进制日志 、 错误日志 、 通用查询日志 和 慢查询日志 ，这也是常用的4种。MySQL 8又新增两种支持的日志： 中继日志 和 数据定义语句日志 。使 用这些日志文件，可以查看MySQL内部发生的事情。

**这6类日志分别为：**

- **慢查询日志：**记录所有执行时间超过long\_query\_time的所有查询，方便我们对查询进行优化。
- **通用查询日志：**记录所有连接的起始时间和终止时间，以及连接发送给数据库服务器的所有指令， 对我们复原操作的实际场景、发现问题，甚至是对数据库操作的审计都有很大的帮助
- **错误日志：**记录MySQL服务的启动、运行或停止MySQL服务时出现的问题，方便我们了解服务器的 状态，从而对服务器进行维护。**
- 二进制日志：**记录所有更改数据的语句，可以用于主从服务器之间的数据同步，以及服务器遇到故 障时数据的无损失恢复。**
- 中继日志：**用于主从服务器架构中，从服务器用来存放主服务器二进制日志内容的一个中间文件。 从服务器通过读取中继日志的内容，来同步主服务器上的操作。**
- **数据定义语句日志：**记录数据定义语句执行的元数据操作。

除二进制日志外，其他日志都是 文本文件 。默认情况下，所有日志创建于 `MySQL数据目录` 中。

## 2 **日志的弊端**

* 日志功能会 `降低MySQL数据库的性能` 

* 日志会 `占用大量的磁盘空间` 

MySQL数据目录 中。

# 慢查询日志(slow query log) 

前面章节《第09章\_性能分析工具的使用》已经详细讲述。

# 通用查询日志(general query log)

通用查询日志用来 `记录用户的所有操作` ，包括启动和关闭MySQL服务、所有用户的连接开始时间和截止 时间、发给 MySQL 数据库服务器的所有 SQL 指令等。当我们的数据发生异常时，**查看通用查询日志， 还原操作时的具体场景**，可以帮助我们准确定位问题。

## **问题场景**

## 查看当前状态

```mysql
mysql> SHOW VARIABLES LIKE '%general%';
+------------------+---------------------------------+
| Variable_name    | Value                           |
+------------------+---------------------------------+
| general_log      | OFF                             |
| general_log_file | /var/lib/mysql/afde2166ebb9.log |
+------------------+---------------------------------+
2 rows in set (0.04 sec)
```



## 启动日志

**方式1：永久性方式** 修改my.cnf或者my.ini配置文件来设置。在[mysqld]组下加入log选项，并重启MySQL服务。格式如下：

```mysql
[mysqld]

general\_log=ON
general\_log\_file=[path[filename]]  #日志文件所在目录路径，filename为日志文件名
```

如果不指定目录和文件名，通用查询日志将默认存储在MySQL数据目录中的hostname.log文件中， hostname表示主机名。

**方式2：临时性方式**

```mysql
SET GLOBAL general_log=on; # 开启通用查询日志
SET GLOBAL general_log_file=’path/filename’; # 设置日志文件保存位置
# 对应的，关闭操作SQL命令如下：
SET GLOBAL general_log=off; # 关闭通用查询日志
#查看设置后情况：
查看设置后情况：
```

## 查看日志

通用查询日志是以 `文本文件` 的形式存储在文件系统中的，可以使用 `文本编辑器` 直接打开日志文件。每台 MySQL服务器的通用查询日志内容是不同的。

- 在Windows操作系统中，使用文本文件查看器；
- 在Linux系统中，可以使用vi工具或者gedit工具查看；
- 在Mac OSX系统中，可以使用文本文件查看器或者vi等工具查看。

从 `SHOW VARIABLES LIKE 'general\_log%'`; 结果中可以看到通用查询日志的位置。

在通用查询日志里面，我们可以清楚地看到，什么时候开启了新的客户端登陆数据库，登录之后做了什 么 SQL 操作，针对的是哪个数据表等信息。

## 停止日志

**方式1：永久性方式**

修改 `my.cnf` 或者 `my.ini` 文件，把[mysqld]组下的 `general\_log` 值设置为 `OFF` 或者把general\_log一项注释掉。修改保存后，再重启MySQL服务 ，即可生效。 举例1：

```mysql
[mysqld] 
general\_log=OFF
```

举例2：

```mysql
[mysqld] 
#general\_log=ON
```

**方式2：临时性方式** 

使用SET语句停止MySQL通用查询日志功能：

```mysql
SET GLOBAL general\_log=off; 
```

查询通用日志功能：

```mysql
SHOW VARIABLES LIKE 'general\_log%';
```



## 删除与刷新日志

如果数据的使用非常频繁，那么通用查询日志会占用服务器非常大的磁盘空间。数据管理员可以删除很 长时间之前的查询日志，以保证MySQL服务器上的硬盘空间。

**手动删除文件**

```mysql
SHOW VARIABLES LIKE 'general\_log%';
```

可以看出，通用查询日志的目录默认为MySQL数据目录。在该目录下手动删除通用查询日志 mysql.log。

使用如下命令重新生成查询日志文件，具体命令如下。刷新MySQL数据目录，发现创建了新的日志文 件。前提一定要开启通用日志。

```mysql
mysqladmin -uroot -p flush-logs
```

# 错误日志(error log)

## 启动日志

在MySQL数据库中，错误日志功能是 `默认开启` 的。而且，错误日志 `无法被禁止` 。

默认情况下，错误日志存储在MySQL数据库的数据文件夹下，名称默认为 `mysqld.log` （Linux系统）或hostname.err （mac系统）。如果需要制定文件名，则需要在my.cnf或者my.ini中做如下配置：

```mysql
[mysqld]
log-error=[path/[filename]]  #path为日志文件所在的目录路径，filename为日志文件名
```

修改配置项后，需要重启MySQL服务以生效。

## **查看日志**

MySQL错误日志是以文本文件形式存储的，可以使用文本编辑器直接查看。 查询错误日志的存储路径：

```mysql
mysql>  SHOW VARIABLES LIKE 'log_err%';
+---------------------+--------------------------------+
| Variable_name       | Value                          |
+---------------------+--------------------------------+
| log_error           | /var/lib/mysql/mysql.error.log |
| log_error_verbosity | 3                              |
+---------------------+--------------------------------+
2 rows in set (0.04 sec)
```

执行结果中可以看到错误日志文件是mysqld.log，位于MySQL默认的数据目录下。

## **删除与刷新日志**

对于很久以前的错误日志，数据库管理员查看这些错误日志的可能性不大，可以将这些错误日志删除， 以保证MySQL服务器上的 `硬盘空间` 。MySQL的错误日志是以文本文件的形式存储在文件系统中的，可以`直接删除` 。

```mysql
[root@atguigu01 log]# mysqladmin -uroot -p flush-logs
Enter password: 
mysqladmin: refresh failed; error: 'Could not open file '/var/log/mysqld.log' for error logging.'
```

官网提示：

![](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202203251744899.jpeg)

补充操作：

```mysql
install -omysql -gmysql -m0644 /dev/null /var/log/mysqld.log
```




# binlog

> 配合备份，恢复数据的日志
>
> 主从复制的前提

## 1 参数

```mysql
show varibales like ''
```

| 参数             | 说明           |
| ---------------- | -------------- |
| log_bin          | 日志开关       |
| log_bin_basename | 路径以及名称   |
| server_id        | 服务ID号       |
| binlog_format    | 二进制日志格式 |
| sync_binlog      | 写盘时机       |
|                  |                |
|                  |                |
|                  |                |
|                  |                |

## 2 命令

```mysql
mysql> show binlog events;   #只查看第一个binlog文件的内容
mysql> show binlog events in'mysql-bin.000002'; #查看指定binlog文件的内容
mysql> show binary logs;  #获取binlog文件列表
mysql> show master status； #查看当前正在写入的binlog文件
```

**show binlog events in'mysql-bin.000002';** 

```ruby
Master [binlog]>show binlog events in 'mysql-bin.000003';
+------------------+-----+----------------+-----------+-------------+----------------------------------------+
| Log_name         | Pos | Event_type     | Server_id | End_log_pos | Info                                   |
+------------------+-----+----------------+-----------+-------------+----------------------------------------+
| mysql-bin.000003 |   4 | Format_desc    |         6 |         123 | Server ver: 5.7.20-log, Binlog ver: 4  |
| mysql-bin.000003 | 123 | Previous_gtids |         6 |         154 |                                        |
| mysql-bin.000003 | 154 | Anonymous_Gtid |         6 |         219 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'   |
| mysql-bin.000003 | 219 | Query          |         6 |         319 | create database binlog                 |
| mysql-bin.000003 | 319 | Anonymous_Gtid |         6 |         384 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'   |
| mysql-bin.000003 | 384 | Query          |         6 |         486 | use `binlog`; create table t1 (id int) |
+------------------+-----+----------------+-----------+-------------+----------------------------------------+

Log_name：binlog文件名
Pos：开始的position    *****
Event_type：事件类型
Format_desc：格式描述，每一个日志文件的第一个事件，多用户没有意义，MySQL识别binlog必要信息
Server_id：mysql服务号标识
End_log_pos：事件的结束位置号 *****
Info：事件内容*****
补充:
SHOW BINLOG EVENTS
   [IN 'log_name']
   [FROM pos]
   [LIMIT [offset,] row_count]
[root@db01 binlog]# mysql -e "show binlog events in 'mysql-bin.000004'" |grep drop

```

 **show master status；**

```mysql
Master [(none)]>show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000003 |      154 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
Master [(none)]>

file：当前MySQL正在使用的文件名
Position：最后一个事件的结束位置号
```

## 3 mysqlbinlog查看日志内容

mysqlbinlog是一个查看mysql二进制日志的工具，可以把mysql上面的所有操作记录从日志里导出，这个工具默认的安装路径为：`/usr/local/mysql/bin/mysqlbinlog`

   可以通过`find / -name "mysqlbinlog`"命令查找mysqlbinlog的工具路径。

> m1 安装 mysql-client 
>
> brew install mysql-client

binlog本身是一类二进制文件。二进制文件更省空间，写入速度更快，是无法直接打开来查看的。

因此mysql提供了命令mysqlbinlog进行查看。

```mysql
# 一般的statement格式的二进制文件，用下面命令就可以
mysqlbinlog mysql-bin.000001
# 如果是row格式，加上-v或者-vv参数就行，如
mysqlbinlog -vv mysql-bin.000001

mysqlbinlog /data/mysql/mysql-bin.000006
mysqlbinlog --base64-output=decode-rows -vvv /data/binlog/mysql-bin.000003
mysqlbinlog  -d binlog /data/binlog/mysql-bin.000003
 mysqlbinlog --start-datetime='2019-05-06 17:00:00' --stop-datetime='2019-05-06 17:01:00'  /data/binlog/mysql-bin.000004 
```

## 4 基于Position号进行日志截取

核心就是找截取的起点和终点

```ruby
--start-position=321
--stop-position=513
 mysqlbinlog --start-position=219 --stop-position=1347 /data/binlog/mysql-bin.000003 >/tmp/bin.sql

案例: 使用binlog日志进行数据恢复
模拟:
1. 
[(none)]>create database binlog charset utf8;
2. 
[(none)]>use binlog;
[binlog]>create table t1(id int);
3. 
[binlog]>insert into t1 values(1);
[binlog]>commit;
[binlog]>insert into t1 values(2);
[binlog]>commit;
[binlog]>insert into t1 values(3);
[binlog]>commit;
4. 
[binlog]>drop database binlog;
恢复:
[(none)]>show master status ;
[(none)]>show binlog events in 'mysql-bin.000004';
[root @db01 binlog]# mysqlbinlog --start-position=1227 --stop-position=2342 /data/binlog/mysql-bin.000004 >/tmp/bin.sql
# 临时关闭二进制记录
[(none)]>set sql_Log_bin=0;
[(none)]>source /tmp/bin.sql

面试案例:
1. 备份策略每天全备,有全量的二进制日志
2. 业务中一共10个库,其中一个被误drop了
3. 需要在其他9个库正常工作过程中进行数据恢复

```

## 5 GTID

> 5.6 版本新加的特性,5.7中做了加强
> 5.6 中不开启,没有这个功能.
> 5.7 中的GTID,即使不开也会有自动生成
> SET @@SESSION.GTID_NEXT= 'ANONYMOUS'

```mysql
vim /etc/my.cnf
gtid-mode=on
enforce-gtid-consistency=true
systemctl restart mysqld

Master [(none)]>create database gtid charset utf8;
Query OK, 1 row affected (0.01 sec)

Master [(none)]>show master status ;
+------------------+----------+--------------+------------------+----------------------------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set                      |
+------------------+----------+--------------+------------------+----------------------------------------+
| mysql-bin.000004 |      326 |              |                  | dff98809-55c3-11e9-a58b-000c2928f5dd:1 |
+------------------+----------+--------------+------------------+----------------------------------------+
1 row in set (0.00 sec)

Master [(none)]>use gtid
Database changed
Master [gtid]>create table t1 (id int);
Query OK, 0 rows affected (0.01 sec)

Master [gtid]>show master status ;
+------------------+----------+--------------+------------------+------------------------------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set                        |
+------------------+----------+--------------+------------------+------------------------------------------+
| mysql-bin.000004 |      489 |              |                  | dff98809-55c3-11e9-a58b-000c2928f5dd:1-2 |
+------------------+----------+--------------+------------------+------------------------------------------+
1 row in set (0.00 sec)

Master [gtid]>create table t2 (id int);
Query OK, 0 rows affected (0.01 sec)

Master [gtid]>create table t3 (id int);
Query OK, 0 rows affected (0.02 sec)

Master [gtid]>show master status ;
+------------------+----------+--------------+------------------+------------------------------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set                        |
+------------------+----------+--------------+------------------+------------------------------------------+
| mysql-bin.000004 |      815 |              |                  | dff98809-55c3-11e9-a58b-000c2928f5dd:1-4 |
+------------------+----------+--------------+------------------+------------------------------------------+
1 row in set (0.00 sec)

Master [gtid]>begin;
Query OK, 0 rows affected (0.00 sec)

Master [gtid]>insert into t1 values(1);
Query OK, 1 row affected (0.00 sec)

Master [gtid]>commit;
Query OK, 0 rows affected (0.00 sec)

Master [gtid]>show master status ;
+------------------+----------+--------------+------------------+------------------------------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set                        |
+------------------+----------+--------------+------------------+------------------------------------------+
| mysql-bin.000004 |     1068 |              |                  | dff98809-55c3-11e9-a58b-000c2928f5dd:1-5 |
+------------------+----------+--------------+------------------+------------------------------------------+
1 row in set (0.00 sec)

Master [gtid]>begin;
Query OK, 0 rows affected (0.00 sec)

Master [gtid]>insert into t2 values(1);
Query OK, 1 row affected (0.00 sec)

Master [gtid]>commit;
Query OK, 0 rows affected (0.01 sec)

Master [gtid]>show master status ;
+------------------+----------+--------------+------------------+------------------------------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set                        |
+------------------+----------+--------------+------------------+------------------------------------------+
| mysql-bin.000004 |     1321 |              |                  | dff98809-55c3-11e9-a58b-000c2928f5dd:1-6 |
+------------------+----------+--------------+------------------+------------------------------------------+
1 row in set (0.00 sec)
 
```

### 5.1 基于GTID进行查看binlog

```ruby
具备GTID后,截取查看某些事务日志:
--include-gtids
--exclude-gtids
mysqlbinlog --include-gtids='dff98809-55c3-11e9-a58b-000c2928f5dd:1-6' --exclude-gtids='dff98809-55c3-11e9-a58b-000c2928f5dd:4'  /data/binlog/mysql-bin.000004
```

### 5.2  GTID的幂等性

```ruby
开启GTID后,MySQL恢复Binlog时,重复GTID的事务不会再执行了
就想恢复?怎么办?
--skip-gtids 告诉系统重新生成gtid防止恢复时候，幂等原因无法恢复
mysqlbinlog --include-gtids='3ca79ab5-3e4d-11e9-a709-000c293b577e:4' /data/binlog/mysql-bin.000004 /data/binlog/mysql-bin.000004
set sql_log_bin=0;
source /tmp/binlog.sql
set sql_log_bin=1;
```

### 5.3 使用二进制日志恢复数据案例

#### 5.3.1 故障环境介绍

```ruby
创建了一个库  db, 导入了表t1 ,t1表中录入了很多数据
一个开发人员,drop database db;
没有备份,日志都在.怎么恢复?
思路:找到建库语句到删库之前所有的日志,进行恢复.(开启了GTID模式)
故障案例模拟:
(0) drop database if exists db ;
(1) create database db charset utf8;     
(2) use db;
(3) create table t1 (id int);
(4) insert into t1 values(1),(2),(3);
(5) insert into t1 values(4),(5),(6);
(6) commit
(7) update t1 set id=30 where id=3;
(8) commit;
(9) delete from t1 where id=4;
(10)commit;
(11)insert into t1 values(7),(8),(9);
(12)commit;
(13)drop database db;
========================
drop database if exists db ;
create database db charset utf8; 
use db;
create table t1 (id int);
insert into t1 values(1),(2),(3);
insert into t1 values(4),(5),(6);
commit;
update t1 set id=30 where id=3;
commit;
delete from t1 where id=4;
commit;
insert into t1 values(7),(8),(9);
commit;
drop database db;
=======
运行以上语句，模拟故障场景
需求：将数据库恢复到以下状态（提示第9步和第13步是误操作，其他都是正常操作）
```

#### 5.3.2 恢复过程(无GTID时的恢复)

1. 查看当前使用的 binlog文件

```ruby
oldguo [db]>show master status ;
+------------------+----------+--------------+------------------+-------------------+

| File            | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |

+------------------+----------+--------------+------------------+-------------------+

| mysql-bin.000006 |    1873 |              |                  |                  |

+------------------+----------+--------------+------------------+-------------------+

2.查看事件：

第一段：
| mysql-bin.000006 |  813 | Query      |        1 |        907 | use `db`; create table t1 (id int)                  |

| mysql-bin.000006 |  907 | Query      |        1 |        977 | BEGIN                                              |

| mysql-bin.000006 |  977 | Table_map  |        1 |        1020 | table_id: 77 (db.t1)                                |

| mysql-bin.000006 | 1020 | Write_rows  |        1 |        1070 | table_id: 77 flags: STMT_END_F                      |

| mysql-bin.000006 | 1070 | Table_map  |        1 |        1113 | table_id: 77 (db.t1)                                |

| mysql-bin.000006 | 1113 | Write_rows  |        1 |        1163 | table_id: 77 flags: STMT_END_F                      |

| mysql-bin.000006 | 1163 | Xid        |        1 |        1194 | COMMIT /* xid=74 */                                |

| mysql-bin.000006 | 1194 | Query      |        1 |        1264 | BEGIN                                              |

| mysql-bin.000006 | 1264 | Table_map  |        1 |        1307 | table_id: 77 (db.t1)                                |

| mysql-bin.000006 | 1307 | Update_rows |        1 |        1353 | table_id: 77 flags: STMT_END_F                      |

| mysql-bin.000006 | 1353 | Xid        |        1 |        1384 | COMMIT /* xid=77 */   

mysqlbinlog --start-position=813 --stop-position=1384 /data/mysql/mysql-bin.000006 >/tmp/bin1.sql 
```

第二段：

```mysql
| mysql-bin.000006 | 1568 | Query      |        1 |        1638 | BEGIN                                              |

| mysql-bin.000006 | 1638 | Table_map  |        1 |        1681 | table_id: 77 (db.t1)                                |

| mysql-bin.000006 | 1681 | Write_rows  |        1 |        1731 | table_id: 77 flags: STMT_END_F                      |

| mysql-bin.000006 | 1731 | Xid        |        1 |        1762 | COMMIT /* xid=81 */ 

mysqlbinlog --start-position=1568 --stop-position=1762 /data/mysql/mysql-bin.000006 >/tmp/bin2.sql
```

恢复

```mysql
set sql_log_bin=0;
source /tmp/bin1.sql
source /tmp/bin2.sql
set sql_log_bin=1;
oldguo [db]>select * from t1;

+------+

| id  |

+------+

|    1 |

|    2 |

|  30 |

|    4 |

|    5 |

|    6 |

|    7 |

|    8 |

|    9 |
```

#### 5.3.3  有GTID的恢复

(1)截取

```shell
mysqlbinlog --skip-gtids --include-gtids='3ca79ab5-3e4d-11e9-a709-000c293b577e:7-12' mysql-bin.000004> /tmp/bin.sql

```

(2)恢复

```mysql
set sql_log_bin=0;
source /tmp/bin.sql
```

## 6 二进制日志其他操作

### 6. 1 自动清理日志

```mysql
show variables like '%expire%';
expire_logs_days  0 表示永不过期

# 自动清理时间,是要按照全备周期+1
# 如果七天一次全备
set global expire_logs_days=8;
永久生效:
my.cnf
expire_logs_days=15;
企业建议,至少保留两个全备周期+1的binlog
也就是14+1=15天

```

### 6.2  手工清理

```mysql
# 清理三天之前的binglog
PURGE BINARY LOGS BEFORE now() - INTERVAL 3 day;
# 删除10之前的
PURGE BINARY LOGS TO 'mysql-bin.000010';
注意:不要手工 rm binlog文件
1. my.cnf binlog关闭掉,启动数据库
2.把数据库关闭,开启binlog,启动数据库
删除所有binlog,并从000001开始重新记录日志

# 危险命令，执行主从必蹦
reset master
# 手工滚动binlog
flush logs 
# 重启也会出发日志滚动
# max_binlog_size 控制滚动
```

# slow_log

> 记录慢SQL语句的日志，定位低效SQL语句工具日志

## 1 开启慢日志(默认没开启)

```
开关:
slow_query_log=1 
文件位置及名字 
slow_query_log_file=/data/mysql/slow.log
设定慢查询时间:
long_query_time=0.1
没走索引的语句也记录:
log_queries_not_using_indexes

vim /etc/my.cnf
slow_query_log=1 
slow_query_log_file=/data/mysql/slow.log
long_query_time=0.1
log_queries_not_using_indexes
systemctl restart mysqld
```

## 2 mysqldumpslow 分析慢日志

```mysql
# -s 指定排序 c 表示count也就是出现次数
# -t top 取出前几条
mysqldumpslow -s c -t 10 /data/mysql/slow.log

# 第三方工具(自己扩展)
https://www.percona.com/downloads/percona-toolkit/LATEST/
yum install perl-DBI perl-DBD-MySQL perl-Time-HiRes perl-IO-Socket-SSL perl-Digest-MD5
toolkit工具包中的命令:
./pt-query-diagest  /data/mysql/slow.log
Anemometer基于pt-query-digest将MySQL慢查询可视化
```

