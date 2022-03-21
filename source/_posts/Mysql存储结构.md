---
title: MySQL存储结构
top: false
cover: false
toc: true
mathjax: true
categories:
  - mysql
tags:
  - mysql
date: 2022-03-21 14:45:01
password:
summary:
---

# MySQL存储结构



## 1 物理存储结构

### 1. 1 数据库文件的存放路径

**MySQL数据库文件的存放路径：/var/lib/mysql/**

```ruby
mysql> show variables like "datadir"
+---------------+-----------------+
| Variable_name | Value           |
+---------------+-----------------+
| datadir       | /var/lib/mysql/ |
+---------------+-----------------+
1 row in set (0.01 sec)
```

### 1.2 相关命令目录

相关命令目录：/usr/bin（mysqladmin、mysqlbinlog、mysqldump等命令）和/usr/sbin。

### 1.3 配置文件目录

```ruby
[root@db01 ~]# mysqld --help --verbose |grep my.cnf
/etc/my.cnf /etc/mysql/my.cnf /usr/local/mysql/etc/my.cnf ~/.my.cnf
注:
默认情况下，MySQL启动时，会依次读取以上配置文件，如果有重复选项，会以最后一个文件设置的为准。
但是，如果启动时加入了--defaults-file=xxxx时，以上的所有文件都不会读取.
```

## 2 数据库和文件系统的关系

### 2. 1 查看默认数据库

查看一下在我的计算机上当前有哪些数据库：

```mysql
mysql> SHOW DATABASES;
```

可以看到有 4 个数据库是属于MySQL自带的系统数据库。

* `mysql`  MySQL 系统自带的核心数据库，它存储了MySQL的用户账户和权限信息，一些存储过程、事件的定义信息，一些运行过程中产生的日志信息，一些帮助信息以及时区信息等。

* `information_schema`MySQL 系统自带的数据库，这个数据库保存着MySQL服务器`维护的所有其他数据库的信息`，比如有
  哪些表、哪些视图、哪些触发器、哪些列、哪些索引。这些信息并不是真实的用户数据，而是一些描述性信息，有时候也称之为`元数据`。在系统数据库`information_schema`中提供了一些以`innodb_sys`开头的表，用于表示内部系统表。

* `performance_schema`MySQL 系统自带的数据库，这个数据库里主要保存MySQL服务器运行过程中的一些状态信息，可以
  用来`监控 MySQL 服务的各类性能指标`。包括统计最近执行了哪些语句，在执行过程的每个阶段都花费了多长时间，内存的使用情况等信息。
* `sys` MySQL 系统自带的数据库，这个数据库主要是通过`视图`的形式把`information_schema`和`performance_schema`结合起来，帮助系统管理员和开发人员监控 MySQL 的技术性能。

### 2. 2 数据库在文件系统中的表示

```ruby
❯ ls -all
total 393504
drwxr-xr-x@  29 curiosity  staff       928  3 21 15:54 .
drwxr-xr-x    5 curiosity  staff       160  3 15 17:53 ..
-rw-r-----    1 curiosity  staff         2  3 18 15:47 afde2166ebb9.pid
-rw-r-----    1 curiosity  staff        56  3 18 15:46 auto.cnf
-rw-------    1 curiosity  staff      1680  3 18 15:46 ca-key.pem
-rw-r--r--    1 curiosity  staff      1112  3 18 15:46 ca.pem
-rw-r--r--    1 curiosity  staff      1112  3 18 15:46 client-cert.pem
-rw-------    1 curiosity  staff      1680  3 18 15:46 client-key.pem
-rw-r-----    1 curiosity  staff      1331  3 18 15:47 ib_buffer_pool
-rw-r-----    1 curiosity  staff  50331648  3 21 15:02 ib_logfile0
-rw-r-----    1 curiosity  staff  50331648  3 18 15:46 ib_logfile1
-rw-r-----    1 curiosity  staff  79691776  3 21 15:02 ibdata1
-rw-r-----    1 curiosity  staff  12582912  3 21 14:59 ibtmp1
drwxr-x---   77 curiosity  staff      2464  3 18 15:47 mysql
-rw-r-----    1 curiosity  staff       177  3 18 15:47 mysql-bin.000001
-rw-r-----    1 curiosity  staff       177  3 18 15:47 mysql-bin.000002
-rw-r-----    1 curiosity  staff       539  3 21 15:02 mysql-bin.000003
-rw-r-----    1 curiosity  staff        57  3 18 15:47 mysql-bin.index
-rw-r-----    1 curiosity  staff     14843  3 21 14:58 mysql.error.log
-rw-r-----    1 curiosity  staff   7428294  3 21 16:00 mysql.slow.log
srwxrwxrwx    1 curiosity  staff         0  3 18 15:47 mysql.sock
-rw-------    1 curiosity  staff         2  3 18 15:47 mysql.sock.lock
drwxr-x---   90 curiosity  staff      2880  3 18 15:46 performance_schema
-rw-------    1 curiosity  staff      1680  3 18 15:46 private_key.pem
-rw-r--r--    1 curiosity  staff       452  3 18 15:46 public_key.pem
-rw-r--r--    1 curiosity  staff      1112  3 18 15:46 server-cert.pem
-rw-------    1 curiosity  staff      1680  3 18 15:46 server-key.pem
drwxr-x---  108 curiosity  staff      3456  3 18 15:47 sys
drwxr-x---    5 curiosity  staff       160  3 21 15:02 test
```

这个数据目录下的文件和子目录比较多，除了`information_schema`这个系统数据库外，其他的数据库在`数据目录`下都有对应的子目录。

以我的temp数据库为例，在MySQL 5. 7 中打开：

```ruby

总用量 1144
-rw-r-----. 1 mysql mysql 8658 8 月  18 11 :32 countries.frm
-rw-r-----. 1 mysql mysql 114688 8 月  18 11 :32 countries.ibd
-rw-r-----. 1 mysql mysql 61 8 月  18 11 :32 db.opt
-rw-r-----. 1 mysql mysql 8716 8 月  18 11 :32 departments.frm
-rw-r-----. 1 mysql mysql 147456 8 月  18 11 :32 departments.ibd
-rw-r-----. 1 mysql mysql 3017 8 月  18 11 :32 emp_details_view.frm
-rw-r-----. 1 mysql mysql 8982 8 月  18 11 :32 employees.frm
-rw-r-----. 1 mysql mysql 180224 8 月  18 11 :32 employees.ibd
-rw-r-----. 1 mysql mysql 8660 8 月  18 11 :32 job_grades.frm
-rw-r-----. 1 mysql mysql 98304 8 月  18 11 :32 job_grades.ibd
-rw-r-----. 1 mysql mysql 8736 8 月  18 11 :32 job_history.frm
-rw-r-----. 1 mysql mysql 147456 8 月  18 11 :32 job_history.ibd
-rw-r-----. 1 mysql mysql 8688 8 月  18 11 :32 jobs.frm
-rw-r-----. 1 mysql mysql 114688 8 月  18 11 :32 jobs.ibd
-rw-r-----. 1 mysql mysql 8790 8 月  18 11 :32 locations.frm
-rw-r-----. 1 mysql mysql 131072 8 月  18 11 :32 locations.ibd
-rw-r-----. 1 mysql mysql 8614 8 月  18 11 :32 regions.frm
-rw-r-----. 1 mysql mysql 114688 8 月  18 11 :32 regions.ibd
```

### 2. 3 表在文件系统中的表示

#### 2. 3. 1 InnoDB存储引擎模式

##### 1. 表结构

为了保存表结构，InnoDB在数据目录下对应的数据库子目录下创建了一个专门用于描述表结构的文件，文件名是这样：

```
表名.frm
```

比方说我们在test数据库下创建一个名为  test 的表：

```mysql
mysql> USE test; 
Database changed
mysql> CREATE TABLE test ( 
   ->     c1 INT
   -> );
Query OK, 0 rows affected (0.03 sec)
```

那在数据库  test 对应的子目录下就会创建一个名为  test.frm 的用于描述表结构的文件。.frm文件的格式在不同的平台上都是相同的。这个后缀名为.frm是以二进制格式存储的，我们直接打开是乱码的

##### 2 2. 表中数据和索引

##### ① 系统表空间（system tablespace)

默认情况下，InnoDB会在数据目录下创建一个名为ibdata1、大小为12M的文件，这个文件就是对应的系统表空间在文件系统上的表示。怎么才 12 M？注意这个文件是自扩展文件，当不够用的时候它会自
己增加文件大小。

当然，如果你想让系统表空间对应文件系统上多个实际文件，或者仅仅觉得原来的ibdata1这个文件名难听，那可以在MySQL启动时配置对应的文件路径以及它们的大小，比如我们这样修改一下my.cnf 配置文件

```mysql
[server]
innodb_data_file_path=data1:512M;data2:512M:autoextend
```

##### ② 独立表空间(file-per-table tablespace)

在MySQL5.6.6以及之后的版本中，InnoDB并不会默认的把各个表的数据存储到系统表空间中，而是为`每一个表建立一个独立表空间`，也就是说我们创建了多少个表，就有多少个独立表空间。使用独立表空间来存储表数据的话，会在该表所属数据库对应的子目录下创建一个表示该独立表空间的文件，文件名和表名相同，只不过添加了一个`.ibd`的扩展名而已，所以完整的文件名称长这样：

```
表明.ibd
```

比如我们使用了`独立表空间`去存储test数据库下的test表的话，那么在该表所在数据库对应的test目录下会为test表创建这两个文件夹

```
test.frm
test.ibd
```

其中`test.ibd` 文件就用来存储test标的数据和索引

##### ③ 系统表空间与独立表空间的设置

我们可以自己指定使用 `系统表空间` 还是 `独立表空间` 来存储数据，这个功能由启动参数`innodb_file_per_table` 控制，比如说我们想刻意将表数据都存储到 系统表空间 时，可以在启动MySQL服务器的时候这样配置：

```bash
[server] 
innodb_file_per_table=0 # 0：代表使用系统表空间； 1：代表使用独立表空间
```

默认情况：

```ruby
mysql> show variables like 'innodb_file_per_table'; +-----------------------+-------+
| Variable_name | Value | +-----------------------+-------+
| innodb_file_per_table | ON | +-----------------------+-------+
1 row in set (0.01 sec)
```

##### ④ 其他类型的表空间

随着MySQL的发展，除了上述两种老牌表空间之外，现在还新提出了一些不同类型的表空间，比如通用表空间（generaltablespace）、临时表空间（temporary tablespace）等。

#### 2. 3. 2 MyISAM存储引擎模式

##### 1. 表结构

在存储表结构方面，MyISAM和InnoDB一样，也是在数据目录下对应的数据库子目录下创建了一个专门用于描述表结构的文件：

##### 2. 表中数据和索引

在MyISAM中的索引全部都是二级索引，该存储引擎的数据和索引是分开存放的。所以在文件系统中也是使用不同的文件来存储数据文件和索引文件，同时表数据都存放在对应的数据库子目录下。假如test表使用MyISAM存储引擎的话，那么在它所在数据库对应的atguigu目录下会为test表创建这三个文件：

```mysql
test.frm 存储表结构
test.MYD 存储数据 (MYData)
test.MYI 存储索引 (MYIndex)
```

### 2. 4 小结

举例：`数据库a`，`表b`。

 1 、如果表b采用`InnoDB`，data\a中会产生 1 个或者 2 个文件：

* b.frm ：描述表结构文件，字段长度等

* 如果采用系统表空间模式的，数据信息和索引信息都存储在ibdata1中

* 如果采用独立表空间存储模式，data\a中还会产生b.ibd文件（存储数据信息和索引信息）

此外：

① MySQL5.7 中会在data/a的目录下生成`db.opt`文件用于保存数据库的相关配置。比如：字符集、比较规则。而MySQL8.0不再提供db.opt文件。

② MySQL8.0中不再单独提供b.frm，而是合并在b.ibd文件中。

2 、如果表b采用`MyISAM`，data\a中会产生 3 个文件：

*  MySQL5.7 中：b.frm：描述表结构文件，字段长度等。

   MySQL8.0 中 b.xxx.sdi：描述表结构文件，字段长度等

* b.MYD(MYData)：数据信息文件，存储数据信息(如果采用独立表存储模式)

* b.MYI(MYIndex)：存放索引信息文件

# MySQL 安装目录的目录结构

| 参数         | 路径                                                       | 解释                                     |
| ------------ | ---------------------------------------------------------- | ---------------------------------------- |
| --basedir    | /usr/bin                                                   | 相关命令目录 mysqladmin、mysqldump等命令 |
| --datadir    | /var/lib/mysql/                                            | MySQL 数据库文件的存放路径               |
| --plugin-dir | /usr/lib64/mysql/plugin                                    | MySQL 插件存放路径                       |
| --log-error  | **/var/log/mysqld.log**                                    | MySQL 日志路径                           |
| --pid-file   | /var/run/mysqld/mysqld.pid                                 | 进程 pid 文件                            |
| --socket     | /var/lib/mysql/mysql.sock                                  | 本地连接时用的unix套接字文件             |
|              | **/usr/share/mysql**                                       | 配置文件目录 MySQL 脚本及配置文件        |
|              | /etc/systemd/system/multi-user.target.wants/mysqld.service | 服务启停相关脚本                         |
|              | **/etc/my.cnf**                                            | MySQL 配置文件                           |

# InnoDB数据存储结构

## 1 数据的存储结构页

索引结构给我们提供了搞笑的索引方式，不过索引信息以及数据记录都是保存在文件上的，确切的说是存储在页结构中。另一方面，索引实在存储引擎中实现的，MySQL服务器上的存储引擎负责对表中数据的读取和写入工作。不同的存储引擎存放的格式一般是不同的，甚至有的存储引擎比如Merrory都不用磁盘来存储数据.

### 1.1 磁盘与内存交互的基本单位: 页

InnoDB将数据划分为若干个页,InnoDb中页默认大小为16kb

以`页`作为磁盘和内存之间交互的`基本单位`，也就是一次最少从磁盘中读取16KB的内容到内存中，一次最少把内存中的16KB内容刷新到磁盘中。也就是说，`在数据库中，不论读一行，还是读多行，都是将这些行所在的页进行加载。也就是说，数据库管理存储空间的基本单位是页（Page）`，`数据库/0操作的最小单位是页`。一个页中可以存储多个行记录。

> 记录是按照行来存储的，但是数据库的读取并不以行为单位，否则一次读取（也就是一次IO）只能处理一行数据，效率是很低的

![image-20220321171509748](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202203211715914.png)

### 1.2 页结构概述

页a、页b、页c…页这坐页可以`不在物理结构上相连`，只要通过`双向链表`相关联即可。每个数据页中的记录会按照主键值从小到大的顺组成一个`单向链表`，每个数据页都余为存储在它里边的记录生成一个页目录，在通过主键查找某条记录的时候可以在页目录中`使用二分法`快速定位到对应的槽，然后再遍历该槽对应分组中的记录即可快速找到指定的记灵。

### 1.3页的大小
不同的数据库管理系统（简称DBMS）的页大小不同。比如在MySQL的noDB存储引擎中，默认页的大小是`16KB`，我们可以通过下面的命令来进行查看：

```mysql
mysql>show variables like '%innodb_page_size%';
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| innodb_page_size | 16384 |
+------------------+-------+
1 row in set (0.03 sec)
```

### 1.4页的上层结构
另外在数据库中，还存在着区（Extent）、段（Segment）和表空间（Tablespace）的概念。行、页、区、段、表空间的关系如下图所示：

![image-20220321173033615](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202203211730740.png)

区（Extent）是比页大一级的存储结构，在InnoDB存储引擎中，一个区会分配`64个连续的页`。因为InnoDB中的页大小默认是16KB，所以一个区的大小是64*16KB=`1MB`。

段（Segment）由一个或多个区组成，区在文件系统是一个连续分配的空间（在InnoDB中是连续的64个页）不过在段中不要求区与区之间是相邻的。`段是数据库中的分配单位，不同类型的数据库对象以不同的段形式存在`。当我们创建数据表、索引的时候，就会相应创建对应的段，比如创建一张表时会创建一个表段，创建一个索引时会创建一个索引段。

表空间（Tablespace）是一个逻辑容器，表空间存储的对象是段，在一个表空间中可以有一个或多个段，但是一个段只能属于一个表空间。数据库由一个或多个表空间组成，表空间从管理上可以划分为==系统表空间==、==用户表空间==、==撤销表空间==、==临时表空间==等。

## 2.页的内部结构

页如果按类型划分的话，常见的有`数据页（保存B+树节点）`、`系统页`、`Undo页`和`事务数据页`等。数据页是我们最常使用的页。

数据页的`16KB`大小的存储空间被划分为七个部分，分别是文件头（File Header））、页头（Page Header）、最大最小记录（Infimum-+supremum）、用户记录（User Records）、空闲空间（Free Space）、页目录（PageDirectory）和文件尾（File Tailer）。

页结构的示意图如下所示：

![image-20220321173653368](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202203211736465.png)

![image-20220321173728052](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202203211737142.png)

我们可以把这7个结构分成3个部分。

### 第1 部分：File Header（文件头部）和File Trailer（文件尾部）

首先是`文件通用部分`，也就是`文件头`和`文件尾`。

