---
title: SQL案例分析
top: false
cover: false
toc: true
mathjax: true
categories:
  - SQL案例分析
tags:
  - SQL案例分析
date: 2022-03-30 23:28:53
password:
summary:
---
# 1 直播间人气值

**问题描述**

直播开播记录表t1包含以下字段

*  主播id：author_id

* 直播间id: live_id
* 开播时长: live_duration

直播观看记录表t2包含以下字段

* 观众id:user_id
* 直播间id: live_id
* 观看时长: watching_duration

acu 为平均同时在线人数，计算方式为：观众观看时长/某场直播的开播时长，没有人观看的时候显示为0 
```mysql
# 开播记录表
CREATE TABLE t1(
  author_id integer ,
  live_id integer,
  live_duration integer
  )
  INSERT INTO t1 values(1,1,60),(2,2,120),(3,3,60)
# 观看记录表
CREATE TABLE t2(
  user_id integer ,
  live_id integer,
  watching_duration integer
  )
INSERT INTO t2 values(11,1,60),(12,1,30),(13,1,60)
INSERT INTO t2 values(12,2,60),(14,2,90)
```
  要求计算直播间的人气值输入结果如下

| 主播idd | 直播间i | acu  |
| ------- | ------- | ---- |



```mysql
select t1.author_id,t1.live_id,coalesce(sum(t2.watching_duration/t1.live_duration)) as acu
from t1
left join t2 on (t1.live_id=t2.live_id)
group by t1.author_id,t1.live_id
```

# 2 累计销售金额

**问题描述**

销售记录表t3包含以下字段

| 名称     | 字段名   |
| -------- | -------- |
| 用户ID   | user_id  |
| 销售日志 | sell_day |
| 销售金额 | amount   |

要求机算每个用户首次销售日期后30天内累计销售金额

输出结果格式如下

| 用户id | 首次销售日期 | 30天内累计销售金额 |
| ------ | ------------ | ------------------ |

 ```mysql
 create table t3 (
     user_id integer ,
     sell_day date,
     amount numeric
 )
     insert into t3 (1,'2021-01-01',100),(1,'2021-01-30',100),(1,'2021-02-01',100)
     insert into t3 (2,'2021-01-10',200),(2,'2021-01-11',200),(2,'2021-01-12',200)
 ```

```mysql
# 1 计算首次销售日期
select user_id.min(sell_day) as first_day
from t3
group by user_id

# 2 计算三十天内销售金额
select s.user_id.s.first_day,sum(t3.amount) from t3 
(select user_id,min(sell_day) as first_day
from t3
group by user_id)s
where t3.user_id=s.user_id
and t3.sell_day between s.first_day and s.first_day + interval '30' day

```

