---
title: Mysql DML条件语句与多表操作
top: false
cover: false
toc: true
mathjax: true
categories:
  - Mysql - 多表操作
tags:
  - Mysql - DML条件语句 
date: 2020-12-02 10:32:00
password:
summary:
---

# 1 条件与多表语句

## 1 条件插入

```mysql
# 第一种情况插入指定字段
insert into table_name(column1,column2)values(value1,value2);
# 第二种情况插入所有字段:前提条件是字段顺序必须与表中字段顺序一致
insert into table_name values(value1,value2);
# 第三种情况批量插入数据
insert into table_name values (value1,value2),(value1,value2);
# 也可以指定插入批量数据
insert into table_name(column1,column2) values(value1,value2),(value1,value2);
# 第四种情况就是当满足了指定条件时才插入数据
insert into (column1,column2) select value1,value2 from table_name where ...
# 也就是后面select子句中查询出来的列作为前面的值插入到表中，但是这个列的个数要和前面的字段个数一致。select子句就可以随便写了。
# 例如
insert into role_menu(id,menuId) select '1',menuId from Menu where menuId=1 and isLead='1';
```

## 2  多表删除

### 1 基本删除

```mysql
# 基本删除，从数据表1中删除匹配的的记录
delete t1 
from t1 
inner join t2 
on t1.id=t2.id 
where t1.name is null
# 或者
delete t1 
from t1 
inner join t2 
using(id)
where t1.name is null
# 或者 这里的using 表示要删除的表
DELETE FROM t1 USING t1，t2 WHERE t1.id=t2.id
# order 与 limit 删除
DELETE FROM somelog WHERE user = 'jcole'
ORDER BY timestamp_column LIMIT 1;
```

**小贴士：**

>  用using关键字进行简化。
>    	**1.**查询必须是等值连接。
>   	 **2**.等值连接中的列必须具有相同的名称和数据类型。

### 2 多表删除

##### 1 外键约束

给表新增约束外键并使用级联(cascade)方式对表与表之间关系进行约束。具体的SQL如下:

```mysql
alter table goods_price
add constraint FK_goods_id foreign key(goods_id)
 references goods(id) on delete cascade on update cascade;
```

修改完后我们后期删除主表数据，关联表数据也会被删除。

```mysql
delete from goods where id=5;
```



##### 2 联合删除

不能使用`ORDER BY`或`LIMIT`

```mysql
DELETE t1,t2 from t1 LEFT JOIN t2 ON t1.id=t2.id WHERE t1.id=25
```

**小贴士：**

> ​		对于多表删除的`left join` 、`inner join`、`right join` 和`select` 的一样，删除各个表交集的部分。
>
> ​		个人理解的执行方式就是从t1表取出一条数据，关联t2表，关联的结果通过where进行筛选，最后左右两张表关联的行会被删除。。

![image-20201202171603690](image-20201202171603690.png)

## 3 多表更新

> ​		MySQL可以在一个SQL语句中更新多张表的记录，也可以通过多个表之间的关联关系更新某个表的数据。
>
> ​		假定目前有两张表`goods`和`goods_price`表，前者是保存商品的具体信息，后者是保存商品的价格，具体的表结构如下：

```mysql
create table goods (
`id` int unsigned primary key auto_increment,
`goods_name` varchar(30) not null default '',
`deleted_at` int unsigned default null
)engine innodb charset utf8;
create table goods_price (
`goods_id` int unsigned not null,
`price` decimal(8,2) not null default '0.00'
)engine innodb charset utf8;
insert into goods (id,goods_name) values (1,'商品1'),(2,'商品2'),(3,'商品3'),(4,'商品4'),(5,'商品5');
insert into goods_price values (1,'5.44'),(2,'3.22'),(3,'5.55'),(4,'0.00'),(5,'4.54');
```

### 1 在update时使用逗号分割更新

```mysql
UPDATE goods as g , goods_price as p 
SET p.price = p.price*0.5 
WHERE p.goods_id = g.id AND g.deleted_at is null;
```

### 2 使用inner join更新数据

```mysql
UPDATE goods g 
INNER JOIN goods_price p 
ON g.id=p.goods_id 
SET p.price=p.price*0.5 
where g.deleted_at is null;
```

### 3 更新多个表

上面的更新语句使用另一个表的条件，更新一张表，也可以更新多个表。具体SQL语句如下：

```mysql
UPDATE goods g 
INNER JOIN goods_price p on g.id=p.goods_id 
set p.price=p.price*0.5,g.deleted_at=unix_timestamp(now())
where g.deleted_at is null;
```