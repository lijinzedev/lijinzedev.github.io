---
title: MySQL数据恢复与备份
top: false
cover: false
toc: true
mathjax: true
categories:
  - mysql
  - 运维  
tags:
  - mysql
  - 运维
date: 2021-11-21 20:16:57
password:
summary:
---

# 备份职责

## 1.1 设计备份策略

* 全备  
* 增量
* 时间
* 自动

## 1.2 日常备份检查

```
备份存在性
备份空间够用否
```

## 1.3 定期恢复演练(测试库)

> 恢复演练可以使用脚本

```
一季度 或者 半年
```

## 1.4 故障恢复

```
通过现有备份,能够将数据库恢复到故障之前的时间点.       
```

## 1.5 迁移

```undefined
1. 停机时间
2. 回退方案
```

# MySQL数据备份与恢复

##   备份的方式

```undefined
备份方式：
全备:全库备份，备份所有数据
增量:备份变化的数据
逻辑备份=mysqldump+mysqlbinlog
物理备份=xtrabackup_full+xtrabackup_incr+binlog或者xtrabackup_full+binlog
备份周期:
根据数据量设计备份周期
比如：数据量大 周日全备，周1-周6增量
数据量小，每天全备（100G以内）+ 增量
```

## 1. 备份的简介

1. **备份类型:**
   - 热备份、温备份和冷备份：
     - 热备份：在线备份，读写操作不受影响；只能是innodb
     - 温备份：在线备份，读操作可继续进行，但写操作不允许；（myisam）
     - 冷备份：离线备份，数据库服务器离线，备份期间不能为业务提供读写服务；
   - 物理备份和逻辑备份：
     - 物理备份：直接复制数据文件进行的备份；
     - 逻辑备份：从数据库中“导出”数据另存而进行的备份，与存储引擎无关
2. **规则备份时需要考虑的因素：**
   - 持锁的时长
   - 备份过程时长
   - 备份负载
   - 恢复过程时长
3. **备份什么:**
   - 数据、额外的数据（二进制日志和InnoDB的事务日志）
   - 代码（存储过程和存储函数、触发器、事件调度器等）、服务器配置文件
4. **设计备份方案:**
   - 全量备份+增量备份，binlog
   - 全量备份+差异备份，binlog

## 2.备份工具

逻辑备份工具

```undefined
基于SQL语句进行备份
mysqldump       *****
mysqlbinlog     *****
```

物理备份工具

```undefined
基于磁盘数据文件备份
xtrabackup(XBK) ：percona 第三方   *****
MySQL Enterprise Backup（MEB）
```

1. MySQLdump:

   <!--这是 MySQL 自带的备份工具，其采用的备份方式是逻辑备份，支持全库备份、单库备份、单表备份。由于其采用的是逻辑备份，所以生成的备份文件比物理备份的大，且所需恢复时间也比较长。-->

   - 逻辑备份工具，适用于所有存储引擎，温备；完全备份，部分备份，不支持差异和增量备份
   - 对InnoDB存储引擎支持热备
   - MyISAM 温备

2. cp, tar等文件系统工具: 物理备份工具，适用于所有存储引擎；冷备；完全备份，部分备份；

3. lvm2的快照：请求一个全局锁，之后立即释放，几乎热备

4. MySQLhotcopy: 几乎冷备；仅适用于MyISAM存储引擎；

5. MySQLpump: <!--这是 MySQL 5.7 之后新增的备份工具，在 mysqldump 的基础上进行了功能的扩展，支持多线程备份，支持对备份文件进行压缩，能够提高备份的速度和降低备份文件所需的储存空间。-->

6. xtrabackup: Innodb 热备的物理备份工具，支持全量，增量和差异备份

   <!--这是 Percona 公司开发的实时热备工具，能够在不停机的情况下进行快速可靠的热备份，并且备份期间不会间断数据库事务的处理。它支持数据的全备和增备，并且由于其采用的是物理备份的方式，所以恢复速度比较快。-->

**备份方案工具选择:**

- mysqldump+binlog: mysqldump 完全备份，通过备份二进制日志实现增量备份；
- lvm2快照+binlog：几乎热备，物理备份
- xtrabackup + binlog:
  - 对InnoDB：热备，支持完全备份和增量备份
  - 对MyISAM引擎：温备，只支持完全备份

### 2.1 mysqldump

> 优点：
> 1.不需要下载安装
> 2.备份出来的是SQL，文本格式，可读性高,便于备份处理
> 3.压缩比较高，节省备份的磁盘空间
>
> 缺点：
> 4.依赖于数据库引擎，需要从磁盘把数据读出
> 然后转换成SQL进行转储，比较耗费资源（CPU,内存，IO），数据量大的话效率较低
> 建议：
> 100G以内的数据量级，可以使用mysqldump
> 超过TB以上，我们也可能选择的是mysqldump，配合分布式的系统
> 1EB  =1024 PB =1000000 TB

- 格式:

  - `mysqldump [options] database [tables]`: 单库，多表备份
  - `mysqldump --databases [options] DB1 [DB2....]`: 单库，多库备份
  - `mysqldump --all-databases [options]`: 备份所有库

- 作用: mysql 客户端，通过mysql协议连接至mysqld，支持逻辑，完全，部分备份

- 生成: Schema和数据存储一起保存为巨大的SQL语句、单个巨大的备份文件

- 二次封装工具: mydumper, phpMyAdmin

- 参数:

  - 基础参数 -u  -p   -S   -h  -P    

  ```mysql
  本地备份:
  mysqldump -uroot -p  -S /tmp/mysql.sock
  远程备份:
  mysqldump -uroot -p  -h 10.0.0.51 -P3306
  # --column-statistics=0 为
  mysqldump -u root -p123456 --column-statistics=0 -h 127.0.0.1 -P 13309 -A  >/tmp/mysql_back_test/all.sql
  ```

  - 温备: 支持MyISAM INNODB，MyISAM 必须显示指定

    - -x, –lock-all-tables：锁定所有表，从而保证备份数据的一致性。此选项自动关闭 `--single-transaction` 和 `--lock-tables`。
    - -l, –lock-tables：锁定备份的表，能够保证当前数据库中表的一致性，但不能保证全局的一致性。
  
  - 热备: 支持 INNODB
  
    - –single-transaction：启动一个大的单一事务实现备份（严格说不算是热备，只能说是快照备份）
  
      此选项会将事务隔离模式设置为 REPEATABLE READ 并开启一个事务，从而保证备份数据的一致性。主要用于事务表，如 InnoDB 表。 但是此时仍然不能在备份表上执行 ALTER TABLE， CREATE TABLE， DROP TABLE， RENAME TABLE， TRUNCATE TABLE 等操作，因为 REPEATABLE READ 并不能隔离这些操作。
  
      另外需要注意的是 `--single-transaction` 选项与 `--lock-tables` 选项是互斥的，因为 LOCK TABLES 会导致任何正在挂起的事务被隐式提交。转储大表时，可以将 `--single-transaction` 选项与 `--quick` 选项组合使用 。
  
  - 选库:
  
    - -A, –all-databases
    - -B, –databases db_name1 db_name2 …：备份指定的数据库
    - -C, –compress：压缩传输；
  
  - 其他:
  
    -  --max-allowed-packet=#

      ```csharp
      mysqldump -uroot -p -A -R -E --triggers --master-data=2  --single-transaction --set-gtid-purged=OFF --max-allowed-packet=256M >/data/backup/full.sql
      
       --max-allowed-packet=# 
      The maximum packet length to send to or receive from server.
      ```
  
    - #### set-gtid-purged
  
      - on 主从复制的参数配置
      - off 用在日常备份过程中
      - auto 主从复制的参数配置
  
    - -E, –events：备份事件调度器（计划任务）；
  
    - -R, –routines：备份存储过程和存储函数；
  
    - –triggers：备份触发器
  
    - `--master-data[=#]`
  
      记录备份开始时刻，二进制文件所处的文件和位置，可选值为
  
      - =1：记录CHANGE MASTER TO语句，此语句未被注释；（主从的时候加）
      - =2：记录CHANGE MASTER TO语句，为注释语句，CHANGE MASTER TO 只对从服务有效，通常应该注释掉
  
      可以通过配置此参数来控制生成的备份文件是否包含 CHANGE MASTER 语句，该语句中包含了当前时间点二进制日志的信息。该选项有两个可选值：1 和 2 ，设置为 1 时 CHANGE MASTER 语句正常生成，设置为 2 时以注释的方式生成。
  
      `--master-data` 选项还会自动关闭 `--lock-tables` 选项，而且如果你没有指定 `--single-transaction` 选项，那么它还会启用 `--lock-all-tables` 选项，在这种情况下，会在备份开始时短暂内获取全局读锁。
  
    - –flush-logs, -F：锁定表之后执行flush logs命令，这样二进制日志就会滚动到新的文件，在利用二进制日志进行回滚时就不用进行日志截取了（有几个库就会刷新几个二进制文件）
  
    - –quick， -q：主要用于备份大表。它强制 mysqldump 一次只从服务器检索一行数据，避免一次检索所有行而导致缓存溢出。
  
    - –default-character-set=charset_name: 导出文本使用的字符集，默认为 utf8。
  
    - –ignore-table=db_name.tbl_name
  
      不需要进行备份的表，必须使用数据库和表名来共同指定。也可以作用于视图。
  
    - –where=’where_condition’， -w ‘where_condition’
  
      在对单表进行导出时候，可以指定过滤条件，例如指定用户名 `--where="user='jimf'"` 或用户范围 `-w"userid>1"` 。

#### 2.1.1 全量备份

```mysql

# mysqldump 的全量备份与恢复的操作比较简单，示例如下：
# 备份雇员库
mysqldump  -uroot -p --databases employees > employees_bak.sql

# 恢复雇员库
mysql -uroot -p  < employees_bak.sql

#单表备份：
# 备份雇员库中的职位表
mysqldump  -uroot -p --single-transaction employees titles > titles_bak.sql

# 恢复雇员库中的职位表
mysql> use employees;
mysql> source /root/mysqldata/titles_bak.sql;

# 备份数据库中所有的单库以及单表
select  concat('mysqldump -uroot -p123456 -h127.0.0.1 -P13306 ',
TABLE_SCHEMA,
' ',
TABLE_NAME,
' > /tmp/mysql/',
TABLE_SCHEMA,
'_',
TABLE_NAME,
'.sql'
)
FROM information_schema.TABLES
where TABLE_SCHEMA not in('sys','information_schema','performance_schema')
```

```mysql
# mysqsldump + binlog 做备份的示例
# 1. mysqldump 全量备份
mysqldump -uroot -p --single-transaction -R -E --triggers --master-data=2 --flush-logs --databases tsong > /home/tao/tsong-fullback-$(date +%F).sql

# 2. 将 binlog 生成 sql 语句, 对应的二进制文件已经记录在 mysqldump 内的CHANGE MASTER TO 语句内
sudo mysqlbinlog /data/master-log.000004 > binlog.sql

# 3. 启动新的 mairadb 服务器，执行上述两个 sql 脚本
mysql> SET SESSION sql_log_bin=0;       # 避免重放的 sql 语句记录到新的二进制文件中
mysql> SOURCE /path/from/somefile.sql;  # 重放上述两个 sql 脚本
mysql> SET SESSION sql_log_bin=1;

# 4. 对恢复的数据库重新做一次全量备份
mysqldump -uroot -p --single-transaction -R -E --triggers --master-data=2 --flush-logs --databases tsong > /home/tao/tsong-fullback-$(date +%F).sql
```

#### 2.1.2 增量备份

mysqldump 本身并不能直接进行增量备份，需要通过分析二进制日志的方式来完成。具体示例如下：

##### 1. 基础全备

1.先执行一次全备作为基础，这里以单表备份为例，需要用到上文提到的 `--master-data` 参数，语句如下：

```
mysqldump -uroot -p --master-data=2 --flush-logs employees titles > titles_bak.sql
```

使用 more 命令查看备份文件，此时可以在文件开头看到 CHANGE MASTER 语句，语句中包含了二进制日志的名称和偏移量信息，具体如下：

```
-- CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000004', MASTER_LOG_POS=155;
```

##### 2. 增量恢复

对表内容进行任意修改，然后通过分析二进制日志文件来生成增量备份的脚本文件，示例如下：

```
mysqlbinlog --start-position=155 \
--database=employees  ${MYSQL_HOME}/data/mysql-bin.000004 > titles_inr_bak_01.sql
```

需要注意的是，在实际生产环境中，可能在全量备份后与增量备份前的时间间隔里生成了多份二进制文件，此时需要对每一个二进制文件都执行相同的命令：

```
mysqlbinlog --database=employees  ${MYSQL_HOME}/data/mysql-bin.000005 > titles_inr_bak_02.sql
mysqlbinlog --database=employees  ${MYSQL_HOME}/data/mysql-bin.000006 > titles_inr_bak_03.sql
.....
```

之后将全备脚本 ( titles_bak.sql )，以及所有的增备脚本 ( inr_01.sql，inr_02.sql …. ) 通过 source 命令导入即可，这样就完成了全量 + 增量的恢复。

## 3.mysqlpump

### 3.1 功能优势

mysqlpump 在 mysqldump 的基础上进行了扩展增强，其主要的优点如下：

- 能够并行处理数据库及其中的对象，从而可以加快备份进程；
- 能够更好地控制数据库及数据库对象（表，存储过程，用户帐户等）；
- 能够直接对备份文件进行压缩；
- 备份时能够显示进度指标（估计值）；
- 备份用户时生成的是 CREATE USER 与 GRANT 语句，而不是像 mysqldump 一样备份成数据，可以方便用户按需恢复。

### 3.2 常用参数

mysqlpump 的使用和 mysqldump 基本一致，这里不再进行赘述。以下主要介绍部分新增的可选项，具体如下：

- **–default-parallelism=N**

  每个并行处理队列的默认线程数。默认值为 2。

- **–parallel-schemas=[N:]db_list**

  用于并行备份多个数据库：db_list 是一个或多个以逗号分隔的数据库名称列表；N 为使用的线程数，如果没有设置，则使用 `--default-parallelism` 参数的值。

- **–users**

  将用户信息备份为 CREATE USER 语句和 GRANT 语句 。如果想要只备份用户信息，则可以使用下面的命令：

  ```
  mysqlpump --exclude-databases=% --users
  ```

- **–compress-output=algorithm**

  默认情况下，mysqlpump 不对备份文件进行压缩。可以使用该选项指定压缩格式，当前支持 LZ4 和 ZLIB 两种格式。需要注意的是压缩后的文件可以占用更少的存储空间，但是却不能直接用于备份恢复，需要先进行解压，具体如下：

  ```
  # 采用lz4算法进行压缩
  mysqlpump --compress-output=LZ4 > dump.lz4
  # 恢复前需要先进行解压
  lz4_decompress input_file output_file
    
  # 采用ZLIB算法进行压缩
  mysqlpump --compress-output=ZLIB > dump.zlib
  zlib_decompress input_file output_file
  ```

  MySQL 发行版自带了上面两个压缩工具，不需要进行额外安装。以上就是 mysqlpump 新增的部分常用参数，完整参数可以参考官方文档：[mysqlpump — A Database Backup Program](https://dev.mysql.com/doc/refman/8.0/en/mysqlpump.html#option_mysqlpump_compress-output)

## 4.Xtrabackup

> 优点：
> 1.类似于直接cp数据文件，不需要管逻辑结构，相对来说性能较高
> 缺点：
> 2.可读性差
> 3.压缩比低，需要更多磁盘空间
> 建议：100G<TB

### 备份方式

```ruby
（1）对于非Innodb表（比如 myisam）是，锁表cp数据文件，属于一种温备份。
（2）对于Innodb的表（支持事务的），不锁表，拷贝数据页，最终以数据文件的方式保存下来，把一部分redo和undo一并备走，属于热备方式。
```

###  xbk 在innodb表备份恢复的流程

```ruby
 0、xbk备份执行的瞬间,立即触发ckpt,已提交的数据脏页,从内存刷写到磁盘（保证备份的数据时当前备份时间的）,并记录此时的LSN号
  1、备份时，拷贝磁盘数据页，并且记录备份过程中产生的redo和undo一起拷贝走,也就是checkpoint LSN之后的日志
  2、在恢复之前，模拟Innodb“自动故障恢复”的过程，将redo（前滚）与undo（回滚）进行应用
  3、恢复过程是cp 备份到原来数据目录下
```

### 4.0 innobackupex使用

####  4.0.1 全备

```bash
[root@db01 backup]# innobackupex --user=root --password=123  /data/backup
```

#### 4.0.2 自主定制备份路径名

```bash
[root@db01 backup]# innobackupex --user=root --password=123 --no-timestamp /data/backup/full
```

#### 4.0.3 备份集中多出来的文件：

![文件内容](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202203181715304.png)

```ruby
-rw-r----- 1 root root       24 Jun 29 09:59 xtrabackup_binlog_info
-rw-r----- 1 root root      119 Jun 29 09:59 xtrabackup_checkpoints
-rw-r----- 1 root root      489 Jun 29 09:59 xtrabackup_info
-rw-r----- 1 root root     2560 Jun 29 09:59 xtrabackup_logfile

xtrabackup_binlog_info ：（备份时刻的binlog位置）
[root@db01 full]# cat xtrabackup_binlog_info 
mysql-bin.000003    536749
79de40d3-5ff3-11e9-804a-000c2928f5dd:1-7
记录的是备份时刻，binlog的文件名字和当时的结束的position，可以用来作为截取binlog时的起点。

xtrabackup_checkpoints ：
backup_type = full-backuped 备份类型全备份
from_lsn = 0            上次所到达的LSN号(对于全备就是从0开始,对于增量有别的显示方法)
# 如果差九个就代表to_lsn与last_lsn是一样
to_lsn = 160683027      备份开始时间(ckpt)点数据页的LSN    
last_lsn = 160683036    备份结束后，redo日志最终的LSN
compact = 0
recover_binlog_info = 0

（1）备份时刻，立即将已经commit过的，内存中的数据页刷新到磁盘(CKPT).开始备份数据，数据文件的LSN会停留在to_lsn位置。
（2）备份时刻有可能会有其他的数据写入，已备走的数据文件就不会再发生变化了。
（3）在备份过程中，备份软件会一直监控着redo的undo，如果一旦有变化会将日志也一并备走，并记录LSN到last_lsn。
从to_lsn  ----》last_lsn 就是，备份过程中产生的数据变化.
```

#### 4.0.4 恢复备份

```shell
前提：
1、被恢复的目录是空
2、被恢复的数据库的实例是关闭
systemctl stop mysqld
```

准备备份（Prepared）

```ruby
# 将redo进行重做，已提交的写到数据文件，未提交的使用undo回滚掉。模拟了CSR的过程
innobackupex --apply-log  /backup/full
# 恢复数据
innobackupex --copy-back  /backup/full
# 或者
cp -a /backup/full/* /data/mysql1/

# 更改权限
chown -R mysql.mysql /data/mysql1
#  启动数据库
vim /etc/my.cnf
datadir=/data/mysql1
[root@db01 mysql1]# chown -R mysql.mysql /data/mysql1
systemctl start mysqld
```

#### 4.0.5 增量备份

```csharp
（1）删掉原来备份
略.
（2）全备（周日）
[root@db01 backup]# innobackupex --user=root --password --no-timestamp /backup/full >&/tmp/xbk_full.log
（3）模拟周一数据变化
db01 [(none)]>create database cs charset utf8;
db01 [(none)]>use cs
db01 [cs]>create table t1 (id int);
db01 [cs]>insert into t1 values(1),(2),(3);
db01 [cs]>commit;

（4）第一次增量备份（周一）
innobackupex --user=root --password=123 --no-timestamp --incremental --incremental-basedir=/backup/full  /backup/inc1 &>/tmp/inc1.log
（5）模拟周二数据
db01 [cs]>create table t2 (id int);
db01 [cs]>insert into t2 values(1),(2),(3);
db01 [cs]>commit;
（6）周二增量
 innobackupex --user=root --password=123 --no-timestamp --incremental --incremental-basedir=/backup/inc1  /backup/inc2  &>/tmp/inc2.log
（7）模拟周三数据变化
db01 [cs]>create table t3 (id int);
db01 [cs]>insert into t3 values(1),(2),(3);
db01 [cs]>commit;
db01 [cs]>drop database cs;
```

#### 4.0.6 增量恢复

```kotlin
1. 检查备份
1afe8136-601d-11e9-9022-000c2928f5dd:7-9
2. 备份整理（apply-log）+合并备份（full+inc1+inc2）
(1) 全备的整理
[root@db01 one]# innobackupex --apply-log --redo-only /data/backup/full
(2) 合并inc1到full中
[root@db01 one]# innobackupex --apply-log --redo-only --incremental-dir=/data/backup/inc1 /data/backup/full
(3) 合并inc2到full中
[root@db01 one]# innobackupex --apply-log  --incremental-dir=/data/backup/inc2 /data/backup/full
(4) 最后一次整理全备
[root@db01 backup]#  innobackupex --apply-log  /data/backup/full
3. 截取周二 23:00 到drop 之前的 binlog 
[root@db01 inc2]# mysqlbinlog --skip-gtids --include-gtids='1afe8136-601d-11e9-9022-000c2928f5dd:7-9' /data/binlog/mysql-bin.000009 >/data/backup/binlog.sql
4. 进行恢复
[root@db01 backup]# mkdir /data/mysql/data2 -p
[root@db01 full]# cp -a * /data/mysql/data2
[root@db01 backup]# chown -R mysql.  /data/*
[root@db01 backup]# systemctl stop mysqld
vim /etc/my.cnf
datadir=/data/mysql/data2
systemctl start mysqld
Master [(none)]>set sql_log_bin=0;
Master [(none)]>source /data/backup/binlog.sql
```

### 4.1 在线安装

Xtrabackup 可以直接使用 yum 命令进行安装，这里我的 MySQL 为 8.0 ，对应安装的 Xtrabackup 也为 8.0，命令如下：

```shell
# 安装Percona yum 源
yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm
# 安装
yum install percona-xtrabackup-80
```

### 4.2 全量备份

全量备份的具体步骤如下：

#### 1. 创建备份

Xtrabackup 全量备份的基本语句如下，可以使用 target-dir 指明备份文件的存储位置，parallel 则是指明操作的并行度：

```
xtrabackup --backup  --user=root --password --parallel=3  --target-dir=/data/backups/
```

以上进行的是整个数据库实例的备份，如果需要备份指定数据库，则可以使用 –databases 进行指定。

另外一个容易出现的异常是：Xtrabackup 在进行备份时，默认会去 `/var/lib/mysql/mysql.sock` 文件里获取数据库的 socket 信息，如果你修改了数据库的 socket 配置，则需要使用 –socket 参数进行重新指定，否则会抛出找不到连接的异常。备份完整后需要立即执行的另外一个操作是 prepare （准备备份）。

#### 2. 准备备份

由于备份是将所有物理库表等文件复制到备份目录，而整个过程需要持续一段时间，此时备份的数据中就可能会包含尚未提交的事务或已经提交但尚未同步至数据文件中的事务，最终导致备份结果处于不一致状态。此时需要进行 prepare 操作来回滚未提交的事务及同步已经提交的事务至数据文件，从而使得整体达到一致性状态。命令如下：

```shell
xtrabackup --prepare --target-dir=/data/backups/
```

需要特别注意的在该阶段不要随意中断 xtrabackup 进程，因为这可能会导致数据文件损坏，备份将无法使用。

#### 3. 恢复备份

由于 xtrabackup 执行的是物理备份，所以想要进行恢复，必须先要停止 MySQL 服务。同时这里我们可以删除 MySQL 的数据目录来模拟数据丢失的情况，之后使用以下命令将备份文件拷贝到 MySQL 的数据目录下：

```
# 模拟数据异常丢失
rm -rf /usr/app/mysql-8.0.17/data/*

# 将备份文件拷贝到 data 目录下
xtrabackup --copy-back --target-dir=/data/backups/
```

copy-back 命令只需要指定备份文件的位置，不需要指定 MySQL 数据目录的位置，因为 Xtrabackup 会自动从 `/etc/my.cnf` 上获取 MySQL 的相关信息，包括数据目录的位置。如果不需要保留备份文件，可以直接使用 `--move-back` 命令，代表直接将备份文件移动到数据目录下。此时数据目录的所有者通常为执行命令的用户，需要更改为 mysql 用户，命令如下：

```
chown -R mysql:mysql /usr/app/mysql-8.0.18/data
```

再次启动即可完成备份恢复。

### 4.3 增量备份

使用 Xtrabackup 进行增量备份时，每一次增量备份都需要以上一次的备份为基础，之后再将增量备份运用到第一次全备之上，从而完成备份。具体操作如下：

#### 1. 创建备份

这里首先创建一个全备作为基础：

```
xtrabackup --backup --user=root --password=xiujingmysql. --host=172.17.0.4 --port=13306 --datadir=/data/mysql/data --parallel-3 --target-dir=/data/backups
```

之后修改库中任意数据，然后进行第一次增量备份，此时需要使用 `incremental-basedir` 指定基础目录为全备目录：

```
xtrabackup  --user=root --password --backup  --target-dir=/data/backups/inc1 \
--incremental-basedir=/data/backups/base
```

再修改库中任意数据，然后进行第二次增量备份，此时需要使用 `incremental-basedir` 指定基础目录为上一次增备目录：

```
xtrabackup  --user=root --password --backup  --target-dir=/data/backups/inc2 \
--incremental-basedir=/data/backups/inc1
```

#### 2. 准备备份

准备基础备份：

```
xtrabackup --prepare --apply-log-only --target-dir=/data/backups/base
```

将第一次备份作用于全备数据：

```
xtrabackup --prepare --apply-log-only --target-dir=/data/backups/base \
--incremental-dir=/data/backups/inc1
```

将第二次备份作用于全备数据：

```
xtrabackup --prepare --target-dir=/data/backups/base \
--incremental-dir=/data/backups/inc2
```

在准备备份时候，除了最后一次增备外，其余的准备命令都需要加上 `--apply-log-only` 选项来阻止事务的回滚，因为备份时未提交的事务可能正在进行，并可能在下一次增量备份中提交，如果不进行阻止，那么增量备份将没有任何意义。

#### 3. 恢复备份

恢复备份和全量备份时相同，只需要最终准备好的全备数据复制到 MySQL 的数据目录下即可：

```
xtrabackup --copy-back --target-dir=/data/backups/base
# 必须修改文件权限，否则无法启动
chown -R mysql:mysql /usr/app/mysql-8.0.17/data
```

此时增量备份就已经完成。需要说明的是：按照上面的情况，如果第二次备份之后发生了宕机，那么第二次备份后到宕机前的数据依然没法通过 Xtrabackup 进行恢复，此时就只能采用上面介绍的分析二进制日志的恢复方法。由此可以看出，无论是采用何种备份方式，二进制日志都是非常重要的，因此最好对其进行实时备份。

## 5 二进制日志的备份

想要备份二进制日志文件，可以通过定时执行 cp 或 scp 等命令来实现，也可以通过 mysqlbinlog 自带的功能来实现远程备份，将远程服务器上的二进制日志文件复制到本机，命令如下：

```
mysqlbinlog --read-from-remote-server --raw --stop-never \
--host=主机名 --port=3306 \
--user=用户名 --password=密码  初始复制时的日志文件名
```

需要注意的是这里的用户必须具有 replication slave 权限，因为上述命令本质上是模拟主从复制架构下，从节点通过 IO 线程不断去获取主节点的二进制日志，从而达到备份的目的。

# 故障案例一

## 背景

```csharp
（1）每天全备
（2）binlog日志是完整
（3）模拟白天的数据变化
（4）模拟下午两点误删除数据库
```

## 模拟故障案例并恢复

```mysql
需求： 利用全备+binlog回复数据库误删除之前。
故障模拟及恢复：
1. 模拟周一23:00的全备
mysqldump -uroot -p -A -R -E --triggers --master-data=2  --single-transaction --set-gtid-purged=OFF >/data/backup/full.sql
2. 模拟白天的数据变化
Master [(none)]>create database day1 charset utf8;
Master [(none)]>use day1
Master [day1]>create table t1(id int);
Master [day1]>insert into t1 values(1),(2),(3);
Master [day1]>commit;
Master [world]>update city set countrycode='CHN';
Master [world]>commit;
模拟磁盘损坏：
[root@db01 data]# \rm -rf /data/mysql/data/*
3. 恢复故障
[root@db01 data]# pkill mysqld
# docker 停止mysql
/usr/sbin/mysqld stop

[root@db01 data]# \rm -rf /data/mysql/data/*


```

### 恢复思路

```mysql
1.检查备份可用性
2.从备份中获取二进制日志位置
3.根据日志位置截取需要的二进制日志
4.初始化数据库,并启动
5.恢复全备
6.恢复二进制日志
```



```bash
# 初始化数据库 因为此时磁盘损坏了
mysqld --initialize-insecure --user=mysql --datadir=/data/mysql --basedir=/app/mysql
# 恢复全备
source /data/back/full.sql
# 恢复binlog
source /data/back/binlog.sql
```

## 压缩备份并添加时间戳

```jsx
例子：
mysqldump -uroot -p123 -A  -R  --triggers --master-data=2  --single-transaction|gzip > /backup/full_$(date +%F).sql.gz
mysqldump -uroot -p123 -A  -R  --triggers --master-data=2  --single-transaction|gzip > /backup/full_$(date +%F-%T).sql.gz

mysqldump备份的恢复方式（在生产中恢复要谨慎，恢复会删除重复的表）
set sql_log_bin=0;
source /backup/full_2018-06-28.sql

注意：
1、mysqldump在备份和恢复时都需要mysql实例启动为前提。
2、一般数据量级100G以内，大约15-45分钟可以恢复，数据量级很大很大的时候（PB、EB）
3、mysqldump是覆盖形式恢复的方法。

一般我们认为，在同数据量级，物理备份要比逻辑备份速度快.
逻辑备份的优势:
1、可读性强
2、压缩比很高
```

# 企业故障恢复案例

## 背景环境：

```css
正在运行的网站系统，mysql-5.7.20 数据库，数据量50G，日业务增量1-5M。
```

##  备份策略：

```css
每天23:00点，计划任务调用mysqldump执行全备脚本
```

##  故障时间点：

```undefined
年底故障演练:模拟周三上午10点误删除数据库，并进行恢复.
```

## 思路：

```css
1、停业务，避免数据的二次伤害
2、找一个临时库，恢复周三23：00全备
3、截取周二23：00  --- 周三10点误删除之间的binlog，恢复到临时库
4、测试可用性和完整性
5、 
    5.1 方法一：直接使用临时库顶替原生产库，前端应用割接到新库
    5.2 方法二：将误删除的表导出，导入到原生产库
6、开启业务
处理结果：经过20分钟的处理，最终业务恢复正常
```

## 故障模拟演练

### 准备数据

```csharp
create database backup;
use backup
create table t1 (id int);
insert into t1 values(1),(2),(3);
commit;
rm -rf /backup/*
```

###  周二 23：00全备

```tsx
mysqldump -uroot -p123 -A  -R  --triggers --set-gtid-purged=auto --master-data=2  --single-transaction|gzip > /backup/full_$(date +%F).sql.gz
```

### 模拟周二 23：00到周三 10点之间数据变化

```csharp
use backup
insert into t1 values(11),(22),(33);
commit;
create table t2 (id int);
insert into t2 values(11),(22),(33);
```

### 模拟故障,删除表(只是模拟，不代表生产操作)

```rust
drop database backup;
```

##  恢复过程

###  准备临时数据库（多实例3307）

```undefined
systemctl start mysqld3307
```

### 准备备份

```ruby
（1）准备全备：
cd /backup
gunzip full_2018-10-17.sql.gz 
（2）截取二进制日志
-- CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000036', MASTER_LOG_POS=793;
mysqlbinlog --skip-gtids --include-gtids='3ca79ab5-3e4d-11e9-a709-000c293b577e:6-7' /data/binlog/mysql-bin.000036 >/backup/bin.sql
```

###  恢复备份到临时库

```bash
mysql -S /data/3307/mysql.sock
set sql_log_bin=0;
source /backup/full_2018-10-17.sql
source /backup/bin.sql
```

### 将故障表导出并恢复到生产

```bash
mysqldump   -S /data/3307/mysql.sock backup t1 >/backup/t1.sql
mysql -uroot -p123 
set sql_log_bin=0
use backup 
source /backup/t1.sql;
```



# 课下作业：

```undefined
练习：
1、创建一个数据库 oldboy
2、在oldboy下创建一张表t1
3、插入5行任意数据
4、全备
5、插入两行数据，任意修改3行数据，删除1行数据
6、删除所有数据
7、再t1中又插入5行新数据，修改3行数据
需求，跳过第六步恢复表数据
写备份脚本和策略
```

# 从mysqldump 全备中获取库和表的备份

```kotlin
1、获得表结构
# sed -e'/./{H;$!d;}' -e 'x;/CREATE TABLE `city`/!d;q'  full.sql>createtable.sql

2、获得INSERT INTO 语句，用于数据的恢复

# grep -i 'INSERT INTO `city`'  full.sqll >data.sql &

3.获取单库的备份

# sed -n '/^-- Current Database: `world`/,/^-- Current Database: `/p' all.sql >world.sql
```

# 备份集中单独恢复表

```kotlin
思考:在之前的项目案例中,如果误删除的表只有10M,而备份有500G,该如何快速恢复误删除表?
提示：
drop table city;
create table city like city_bak;
alter table city discard tablespace;
cp /backup/full/world/city.ibd  /application/mysql/data/world/
chown -R mysql.mysql  /application/mysql/data/world/city.ibd 
alter table city import  tablespace;
```

# 反操作语句转换操作

Binlog2sql

https://github.com/danfengcao/binlog2sql

二进制日志反转，针对于dml语句

# 参考

[老男孩](https://www.jianshu.com/p/64906f2dd33f)

