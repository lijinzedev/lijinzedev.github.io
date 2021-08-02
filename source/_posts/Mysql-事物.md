---
title: Mysql 事务
top: false
cover: false
toc: true
mathjax: true
categories:
  - 分类
tags:
  - 标签
date: 2021-07-29 15:49:56
password:
summary:
---

# 一、事物

## 1  事务的属性(ACID)

- 原子性(Atomicity): 原子性是指事物是一个不可分割的单位,事务中的操作要么都发生,要么都不发生
- 一致性(Consistenc): 事务必须使数据库从一个一致性状态变换到另一个一致性状态
- 隔离性(Isolation) 事务的隔离性是指一个事务的执行不能被其他的事务干扰,即一个事务内部的操作及使用的数据库对并发的其他事务是隔离的,并发执行的各个事务之间不能互相干扰.
- 持久性(Durability) : 持久性是指一个事务一旦被提交,它对数据库中的数据的改变就是永久性的,接下来的其他操作和数据库故障不应噶对其有任何影响



## 2 事务隔离级别（tx_isolation）

mysql 有四级事务隔离级别 每个级别都有字符或数字编号

| 级别     | symbol           | 值   | 描述                                                         |
| -------- | ---------------- | ---- | ------------------------------------------------------------ |
| 读未提交 | READ-UNCOMMITTED | 0    | 存在脏读、不可重复读、幻读的问题                             |
| 读已提交 | READ-COMMITTED   | 1    | 解决脏读的问题，存在不可重复读、幻读的问题                   |
| 可重复读 | REPEATABLE-READ  | 2    | mysql 默认级别，解决脏读、不可重复读的问题，存在幻读的问题。使用 MMVC机制 实现可重复读 |
| 序列化   | SERIALIZABLE     | 3    | 解决脏读、不可重复读、幻读，可保证事务安全，但完全串行执行，性能最低 |

我们可以通过以下命令 `查看/设置` `全局/会话` 的事务隔离级别

```ruby
mysql> SELECT @@global.tx_isolation, @@tx_isolation;
+-----------------------+------------------+
| @@global.tx_isolation | @@tx_isolation   |
+-----------------------+------------------+
| REPEATABLE-READ       | READ-UNCOMMITTED |
+-----------------------+------------------+
1 row in set (0.00 sec)

# 设定全局的隔离级别 设定会话 global 替换为 session 即可
# SET [GLOABL] config_name = 'foobar';
# SET @@[session.|global.]config_name = 'foobar';
# SELECT @@[global.]config_name;

SET @@gloabl.tx_isolation = 0;
SET @@gloabl.tx_isolation = 'READ-UNCOMMITTED';

SET @@gloabl.tx_isolation = 1;
SET @@gloabl.tx_isolation = 'READ-COMMITTED';

SET @@gloabl.tx_isolation = 2;
SET @@gloabl.tx_isolation = 'REPEATABLE-READ';

SET @@gloabl.tx_isolation = 3;
SET @@gloabl.tx_isolation = 'SERIALIZABLE';
```

## 3 什么是脏读、幻读、不可重复读

### 1 脏读（Drity Read）

> 某个事物已更新一份数据，另一个事物在此时读取了同一份数据，由于某些原因，前一个RollBack了操作，则后一个事物读取的数据就不会是正确的

### 2 不可重复度(Non-repeatable read)

> 在一个事物的两次查询中数据不一致，这可能是两次查询过程中插入一个事物的更新

### 3 幻读（Phantom Read）

> 在一个事物的两次查询中数据数量不一致，对于两个事务T1,T2,T1从一个表中读取了一个字段,然后T2再该表插入了一些新的行,之后,如果T1再次筛选同一个表,就会多出几行