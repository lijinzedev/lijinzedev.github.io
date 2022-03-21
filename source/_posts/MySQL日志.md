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

