---
title: Redis基础入门
top: false
cover: false
toc: true
mathjax: true
categories:
  - redis
tags:
  - redis
date: 2020-12-26 17:15:44
password:
summary:
---

# Redis 基础入门

## 1 什么是NoSQL

> Not Only SQL

**特点**

* 解耦（数据之间不存在关系）
* 方便扩展
* 大数据量高性能(一秒写八万，读取11万，细颗粒缓存，性能较高)
* 数据类型是多样性的，不需要事先设计数据库，随取随用

**3V+3高**

* 大数据时代的3V，主要描述问题
  * 海量Volume
  * 多样Variety
  * 实时Velocity
* 大数据时代的3高，主要是对程序的要求
  * 高并发
  * 高可扩
  * 高性能

**为什么单线程还这么快**

> 核心：redis是将所有的数据全部放到内存中的，所以说使用单线程去操作效率就是最高的，多线程（CPU上下文会切换会耗时）对于内存系统来说，如果没有上下文切换效率就是最高，多次读写在一个CPU上

## 2 NoSQL的四大分类

**KV键值对**

**文档型数据库（bson格式）**

* NongoDB
  * 基于分布式文件存储的数据库，C++编写，主要处理大量文档
  * 是关系型数据库和非关系型数据库的中间产品，MongoDB是非关系型数据库功能最丰富，最像关系型数据库的
* ConthDB

**列存储数据库**

* HBase
* 分布式文件系统

**图关系数据库**

* 存放关系，如社交网络，广告推荐
* Neo4j，InfoGrid

## 3 Redis

### 1 概述

**Redis是什么**

> Redis是远程字典服务，key-value数据库，并提供多种语言的API,
>
> Redis是单线程的！Redis基于内存操作，CPU不是Redis性能瓶颈，Redis瓶颈是根据机器的内存和网络带宽，

**Redis的功能**

* 内存存储、持久化（rdb,aof）
* 效率高，可以高速缓存
* 发布订阅系统
* 地图信息分析
* 计时器，计数器（微信微博，浏览量）

**Redis的特性**

* 多样的数据类型
* 持久化
* 集群
* 事物

### 2 安装

#### 1 Linux 安装

略

#### 2 Docker 安装

```bash
# 下载镜像
docker pull redis 
# 创建目录
mkdir -p /home/docker/redis/conf
mkdir -p /home/docker/redis/data

# 新增配置文件
cd /home/docker/redis/conf
vim redis.conf
-----------------
# 开启远程访问
#bind 127.0.0.1 
protected-mode no
# 开启数据持久化
appendonly yes
# 设置访问密码
requirepass 123456 
------------------
# 创建容器并启动
docker run --name redis -p 6379:6379 -v /home/docker/redis/data:/data -v /home/docker/redis/conf/redis.conf:/etc/redis/redis.conf -d redis redis-server /etc/redis/redis.conf 

```



### 3 redis-benchmark性能测试

![图片来自菜鸟教程](image-20201226220537474.png)

```bash
# 测试：100个并发连接 100000请求
redis-benchmark -h localhost -p 6379 -c 100 -n 1000000
```

### 4 基础

> redis默认16个数据库（配置文件配置），默认使用第一个数据库可以通过select 切换数据库

```bash
# 切换数据库
127.0.0.1:6379> select 2
# 查看大小
127.0.0.1:6379[2]> DBSIZE
# 查看所有的key
127.0.0.1:6379[2]> keys *
# 清空数据库
127.0.0.1:6379[2]> flushdb
# 清除所有数据库
127.0.0.1:6379[2]> flush moall
# 判断是否存在
EXISTS <key>
# 移除
move <key> 1 (1代表当前数据库)
# 设置过期时间
EXPIRE <key> <seconds>
# 查看当前key的剩余时间
ttl <key>
# 查看当前key的类型
type <key>
```

### 5 数据类型

#### 1 String

```bash
# 追加字符串,如果当前key不存在就创建key
APPPEND <key> <value>
# 查看长度
STRLEN <key>
# 自增命令 自增 1
incr <key> 
# 自减命令 自减 1
decr <key>
# 自增设置步长
INCRBY <key> <increment>
# 自减设置步长
DECRBY <key> <increment>
# 截取字符串 
GETRANGE <key> <start> <end>
# 截取 Hello world 字符串
GETRANGE a 0 4 Hell
# 获取全部字符
GETRANGE a 0 -1
# 替换指定位置开始的字符串
SETRANGE <key> <start> <str>
# setex <> (set with expire ) 设置过期时间
setex 
# setnx (set if not exist) 不存在就设置 (分布式锁中会常常使用)
setnx <k> <v>
# 批量 set (原子性操作)
mset <key> <v1> <k2> <v2>
# 批量 get
mget <k1> <k2> <k3>
# 对象
set user:1 {name:1,age:3} # 设置user对象值位json
# 巧妙的使用mset替换对象对应的字符串
mset user:1:name 1 user:1:age 3
mget user:1:name user:1:age 
# getset 命令 如果存在就返回旧的值,设置新的值,不存在就返回null,设置新的值
```

**应用场景**

* 计数器
* 统计多单位的数量
* 粉丝数
* 对象缓存

#### 2 List

```bash
# 将一个值,或者多个值,插入到表头
LPUSH <key> 1 2 3 
# 将一个值或多个值插入表尾
RPUSH <key> 4 5 6
# 查询所有的值
LRANGE <key> 0 -1
# 左边移除第一个元素
LPOP <key>
# 移除最后一个元素
RPOP <key>
# 根据数组下标获取指定的值
LINDEX <key> <index>
# 获取list长度
LLEN <key>
# 移除指定的值
LREM list <移除个数> <对应value>
LREM list 1 one
# 截取指定返回的list
LTRIM <key> <start> <end>
# 例如 a,b,c,d 截取 2,3 则为 a bs
# 移除并添加命令 从一个list 转移到另一个list
rpoplpush <source> <dest>
# 将列表中指定下标的值替换为新的值,如果不存在key就会报错，存在就没有问题
lset <key> <newVlue>
# 将某一个具体的value，插入list中某个value的前面或者后面
linsert <key> <before|after> <value> <new value>
```



#### 3 Set

```bash
# set集合添加元素
127.0.0.1:6379> sadd myset "hello" "world"
(integer) 2

# 查看set的成员
127.0.0.1:6379> smembers myset
1) "hello"
2) "world"

# 判断指定value在不在set集合中
127.0.0.1:6379> sismember myset hello
(integer) 1
127.0.0.1:6379> 

# 移除set中指定的元素
127.0.0.1:6379> srem myset hello
(integer) 1

# 随机抽选中一个或多个 元素
127.0.0.1:6379> srandmember myset 1
1) "2"
127.0.0.1:6379> srandmember myset 1
1) "1"
127.0.0.1:6379> srandmember myset 1
1) "world"
# 随机删除set集合中的元素
127.0.0.1:6379> spop myset
"3"
# 将一个指定的值移动到另一个结合中
smove myset myset2 "111"
# 交集
sinter <key1> <key2>
# 差集
sdiff <key1> <key2>
# 并集
sunion <key1> <key2> # 共同好友

```

**应用**

* 共同爱好，共同关注，推荐好友

####  4 Hash

> Map集合

```bash
# set
127.0.0.1:6379> hset myhash f1 111 f2 222 f3 333 
(integer) 3

# 获取一个字段值 
127.0.0.1:6379> hget myhash f1
"111"

# 设置多个字段值
127.0.0.1:6379> hmset myhash f4 444
OK

# 获取全部字段值
127.0.0.1:6379> hgetall myhash
1) "f1"
2) "111"
3) "f2"
4) "222"
5) "f3"
6) "333"
7) "f4"
8) "444"

# 删除一个字段值
127.0.0.1:6379> hdel myhash f1
(integer) 1
127.0.0.1:6379> hgetall myhash
1) "f2"
2) "222"
3) "f3"
4) "333"
5) "f4"
6) "444"

# 获取hash的字段数量
127.0.0.1:6379> hlen myhash
(integer) 3

# 判断hash中的key是否存在
127.0.0.1:6379> hexists myhash f1
(integer) 0

# 只获取所有的filed字段
127.0.0.1:6379> hkeys myhash
1) "f2"
2) "f3"
3) "f4"

# 只获取所有的value字段
127.0.0.1:6379> hvals myhash
1) "222"
2) "333"
3) "444"
127.0.0.1:6379> 

# 指定增量
127.0.0.1:6379> hincrby myhash f2 1
(integer) 223

# 如果存在就设置
127.0.0.1:6379> hsetnx myhash f4 1
(integer) 0

```

**应用**

* 用户信息保存
* 适合对象存储

#### 5 Zset

> 有序的set

```bash
# 添加值 
127.0.0.1:6379> zadd myzset 1 a
(integer) 1
127.0.0.1:6379> zadd myzset 2 b
(integer) 1
127.0.0.1:6379> zadd myzset 3 c
(integer) 1
# 显示全部的数据 可以指定范围
127.0.0.1:6379> zrangebyscore myzset -inf +inf 
1) "a"
2) "b"
3) "c"

# 显示小于100的数据
127.0.0.1:6379> zrangebyscore myzset -inf 100 
1) "a"
2) "b"
3) "c"

# 移除元素
127.0.0.1:6379> zrem myzhash f1

# 获取有序集合中的个数
127.0.0.1:6379> zcard myzhash 

# 从大到小排序
127.0.0.1:6379> zrevrange myzhash 0 -1

# 获得指定区间的成员数量
127.0.0.1:6379> zcpunt myzhash 0 3
```

**场景**

* 消息，带权重判断
* 排行榜
* TOP N

#### 6 geospatial

> 地理位置，附近的人

```bash
# 添加经纬度
127.0.0.1:6379> geoadd china:city 116.40 39.40 beijing
(integer) 1
127.0.0.1:6379> geoadd china:city 121.47 31.23 shanghai
(integer) 1
127.0.0.1:6379> geoadd china:city 106.50 29.53 chongqing
(integer) 1
127.0.0.1:6379> geoadd china:city 120.16 30.24 hangzhou
(integer) 1

# 获取指定
127.0.0.1:6379> geopos china:city beijing
   1) "116.39999896287918091"
   2) "39.40000099577971326"

# 定位距离(千米)
geodist china:city beijing shanghai km

# “附近的人” 以给定的经纬度为圆心找到附近的
# 查询110 30 为圆心 半径1000km的key
127.0.0.1:6379> georadius china:city 110 30 1000 km
1) "chongqing"
2) "hangzhou"
# 附加直线距离信息
127.0.0.1:6379> georadius china:city 110 30 1000 km withdist
1) 1) "chongqing"
   2) "341.9374"
2) 1) "hangzhou"
   2) "977.5143"
# 附加经纬度
127.0.0.1:6379> georadius china:city 110 30 1000 km withcoord
# 限制数量
127.0.0.1:6379> georadius china:city 110 30 1000 km withcoord count 1

#  找出指定元素的其他元素
georadiusbymember china：city beijing 1000 km

# 将二维经纬度转化为一位的hash，字符串越接近，距离越近
genhash china:city beijing chongqing

# 使用Zset操作geo
zrange china:city 0 -1

# 移除元素
zrem china:city beijing
```

#### 7 Hyperloglog

> 基数统计。内存小，2^64不同的元素，需要12kb内存，有0.81%的错误率

**什么是基数？**

> 不重复的元素

```bash
# 创建第一组元素
127.0.0.1:6379> pfadd mykey 1 2 3 4 5 6 7 8 9 10
(integer) 1
127.0.0.1:6379> pfadd mykey2 1  3  5  7  9 
(integer) 1
# 统计基数数量
127.0.0.1:6379> pfcount mykey
(integer) 10
127.0.0.1:6379> pfcount mykey2
(integer) 5
# 合并两组
127.0.0.1:6379> pfmerge mykey3 mykey1 mykey2
OK
127.0.0.1:6379> pfcount mykey3
(integer) 5
```

#### 8 Bitmaps

> 位存储，数据结构，操作二进制来进行记录，只有0和1两种状态

**应用**

* 统计用户信息，活跃，不活跃
* 登录，未登录
* 两个状态的

```bash
# 记录打卡状态
127.0.0.1:6379> setbit sign 0 1
(integer) 0
127.0.0.1:6379> setbit sign 1 1
(integer) 0
127.0.0.1:6379> setbit sign 2 0
(integer) 0
# 查看某一天打卡没
127.0.0.1:6379> getbit sign 2
(integer) 0
127.0.0.1:6379> getbit sign 1
(integer) 1
# 统计打卡记录
127.0.0.1:6379> bitcount sign
(integer) 2
```

### 6 事务

> Redis单条命令是保存原子性的，但是事物不保证原子性
>
> Redis事物的本质是一组命令的集合，一个事物中的所有命令都会被序列化，按照顺序执行
>
> 一次性、顺序性、排他行
>
> Redis事物没有隔离级别的概念

**redis事务**

* 开启事务（multi）
* 命令入队(......)
* 执行事务(exec)

#### 1 执行事务

```bash
127.0.0.1:6379> multi 开启事物
OK
127.0.0.1:6379> set 1 1
QUEUED
127.0.0.1:6379> set 2 2
QUEUED
127.0.0.1:6379> set 3 3
QUEUED
127.0.0.1:6379> exec 执行事物
1) OK
2) OK
3) OK
```

#### 2 取消事务

```bash

DISCARD 事务中的队列都不会执行

127.0.0.1:6379> multi
OK
127.0.0.1:6379> set 4 4 
QUEUED
127.0.0.1:6379> set 5 5
QUEUED
127.0.0.1:6379> discard
OK

```

#### 3 命令有错，事务中所有的命令都不会被执行

```bash
127.0.0.1:6379> multi 
OK
127.0.0.1:6379> set 1 1
QUEUED
127.0.0.1:6379> set 2 2
QUEUED
127.0.0.1:6379> set 3 3
QUEUED
127.0.0.1:6379> getset 8
(error) ERR wrong number of arguments for 'getset' command
127.0.0.1:6379> set 5 5
QUEUED
127.0.0.1:6379> exec
(error) EXECABORT Transaction discarded because of previous errors.


```

#### 4 事务存在语法性，执行命令的时候，其他命令是可以正常执行的，错误的命令会抛出异常

```bash
127.0.0.1:6379> set 1 "adasd"
QUEUED
127.0.0.1:6379> incr 1 
QUEUED
127.0.0.1:6379> set 2 2
QUEUED
127.0.0.1:6379> exec
1) OK
2) (error) ERR value is not an integer or out of range
3) OK

```

### 7 乐观锁

> * 乐观，认为什么时候都不会出现问题，所以不会上锁，更新数据的时候去判断一下，在此期间是否有人修改过这个数据
> * 获取version
> * 更新的时候比较version

**正常情况**

```bash
127.0.0.1:6379> set money 100
OK
127.0.0.1:6379> set out 0
OK
127.0.0.1:6379> watch money
OK
127.0.0.1:6379> multi 
OK
127.0.0.1:6379> decrby money 20
QUEUED
127.0.0.1:6379> incrby out 20
QUEUED
127.0.0.1:6379> exec
1) (integer) 80
2) (integer) 20
```

多线程修改值之后，监视失败，使用watch可以当做redis的乐观锁操作

```bash
unwatch  # 如果事物执行失败就先解锁
watch money # 获取最新的值进行监视
```

### 8 Jedis

> 官网推荐的Java连接工具

#### 1 事物

```java
// 创建对象
Jedis jedis = new Jedis("42.193.118.181", 6379);
Transaction multi = jedis.multi();
try {
    jedis.set("user1", "1");
    jedis.set("user1", "2");
    // 执行事物
    multi.exec();
} catch (Exception e) {
    // 放弃事物
    multi.discard();
    e.printStackTrace();
} 
```

### 9 SpringBoot整合

> SpringBoot 2.X之后，jedis被替换了为lettuce
>
> jedis采用直连，多个线程操作的话是不安全的，如果想要避免不安全，使用jedispoolianjiechi1，bio
>
> lettuce：采用netty，实例可以再多个线程中共享，不存在线程不安全的情况，可以减少线程数据NIO

```bash
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

**源码配置**

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(RedisOperations.class)
@EnableConfigurationProperties(RedisProperties.class)
@Import({ LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class })
public class RedisAutoConfiguration {

	@Bean
	@ConditionalOnMissingBean(name = "redisTemplate")
	@ConditionalOnSingleCandidate(RedisConnectionFactory.class)
	public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
		RedisTemplate<Object, Object> template = new RedisTemplate<>();
		template.setConnectionFactory(redisConnectionFactory);
		return template;
	}

	@Bean
	@ConditionalOnMissingBean
	@ConditionalOnSingleCandidate(RedisConnectionFactory.class)
	public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) {
		StringRedisTemplate template = new StringRedisTemplate();
		template.setConnectionFactory(redisConnectionFactory);
		return template;
	}

}

```

**序列化**

> 如果对象不实现Serializable接口直接写入会报错

####  1 自定义RedisTemplate

**略**

### 10 redis.conf详解

#### 1 设置密码

```bash
# 获取密码
config get requirepass
# 设置密码
config set requirepass "123456"
# 登录
auth 123456
```



> redis配置文件对大小写不敏感

```bash


# 不同redis server可以使用同一个模版配置作为主配置，并引用其它配置文件用于本server的个性# 化设置
# include并不会被CONFIG REWRITE命令覆盖。但是主配置文件的选项会被覆盖。
# 想故意覆盖主配置的话就把include放文件前面，否则最好放末尾
# include /path/to/local.conf
# include /path/to/other.conf

######################### 网络 #########################

# 不指定bind的话redis将会监听所有网络接口。这个配置是肯定需要指定的。
# Examples:
# bind 192.168.1.100 10.0.0.1
# bind 127.0.0.1 ::1
# 下面这个配置是只允许本地客户端访问。
bind 127.0.0.1

# 是否开启保护模式。默认开启，如果没有设置bind项的ip和redis密码的话，服务将只允许本地访 问。
protected-mode yes

# 端口设置，默认为 6379
# 如果port设置为0 redis将不会监听tcp socket
port 6379

# 在高并发环境下需要一个高backlog值来避免慢客户端连接问题。注意Linux内核默默将这个值减小到/proc/sys/net/core/somaxconn的值，所以需要确认增大somaxconn和tcp_max_syn_backlog 两个值来达到需要的效果。
tcp-backlog 511

# 指定用来监听Unix套套接字的路径。没有默认值，没有指定的情况下Redis不会监听Unix socket
# unixsocket /tmp/redis.sock
# unixsocketperm 700

# 客户端空闲多少秒后关闭连接（0为不关闭）timeout 0# tcp-keepalive设置。如果非零，则设置SO_KEEPALIVE选项来向空闲连接的客户端发送ACK，用途如下：
# 1）能够检测无响应的对端
# 2）让该连接中间的网络设备知道这个连接还存活
# 在Linux上，这个指定的值(单位秒)就是发送ACK的时间间隔。
# 注意：要关闭这个连接需要两倍的这个时间值。
# 在其他内核上这个时间间隔由内核配置决定# 从redis3.2.1开始默认值为300秒
tcp-keepalive 300

######################### 通用 #########################

# 是否将Redis作为守护进程运行。如果需要的话配置成'yes'
# 注意配置成守护进程后Redis会将进程号写入文件/var/run/redis.pid
daemonize no

# 是否通过upstart或systemd管理守护进程。默认no没有服务监控，其它选项有upstart, systemd, auto
supervised no

# pid文件在redis启动时创建，退出时删除。最佳实践为配置该项。
pidfile /var/run/redis_6379.pid

# 配置日志级别。选项有debug, verbose, notice, warning
loglevel notice

# 日志名称。空字符串表示标准输出。注意如果redis配置为后台进程，标准输出中信息会发送到/dev/null
logfile ""

# 是否启动系统日志记录。
# syslog-enabled no

# 指定系统日志身份。
# syslog-ident redis

# 指定syslog设备。必须是user或LOCAL0 ~ LOCAL7之一。
# syslog-facility local0

# 设置数据库个数。默认数据库是 DB 0
# 可以通过SELECT where dbid is a number between 0 and 'databases'-1为每个连接使用不同的数据库。
databases 16

# 是否显示log
always-show-logo yes

######################### 备份  #########################
# 持久化设置:
# 下面的例子将会进行把数据写入磁盘的操作:
#  900秒（15分钟）之后，且至少1次变更 进行持久化操作
#  300秒（5分钟）之后，且至少10次变更 进行持久化操作
#  60秒之后，且至少10000次变更 进行持久化操作
# 不写磁盘的话就把所有 "save" 设置注释掉就行了。
# 通过添加一条带空字符串参数的save指令也能移除之前所有配置的save指令，如: save ""
save 900 1
save 300 10
save 60 10000

# 默认情况下如果上面配置的RDB模式开启且最后一次的保存失败，redis 将停止接受写操作，让用户知道问题的发生。
# 如果后台保存进程重新启动工作了，redis 也将自动的允许写操作。如果有其它监控方式也可关闭。
# 持久化操作出错了,是否需要继续工作
stop-writes-on-bgsave-error yes

# 是否在备份.rdb文件时是否用LZF压缩字符串，默认设置为yes。如果想节约cpu资源可以把它设置为no。
# 是否压缩rdb文件
rdbcompression yes

# 因为版本5的RDB有一个CRC64算法的校验和放在了文件的末尾。这将使文件格式更加可靠,
# 但在生产和加载RDB文件时，这有一个性能消耗(大约10%)，可以关掉它来获取最好的性能。
# 生成的关闭校验的RDB文件有一个0的校验和，它将告诉加载代码跳过检查rdbchecksum yes
# rdb文件名称
dbfilename dump.rdb

# 保存rdb文件的时候,进行校验检查
rdbchecksum yes

# 备份文件目录，文件名就是上面的 "dbfilename" 的值。累加文件也放这里。
# 注意你这里指定的必须是目录，不是文件名。
dir /Users/wuji/redis_data/

######################### 主从同步 #########################

# 主从同步配置。
# 1) redis主从同步是异步的，但是可以配置在没有指定slave连接的情况下使master停止写入数据。
# 2) 连接中断一定时间内，slave可以执行部分数据重新同步。
# 3) 同步是自动的，slave可以自动重连且同步数据。
# slaveof <masterip> <masterport>

# master连接密码
# masterauth <master-password>

# 当一个slave失去和master的连接，或者同步正在进行中，slave的行为有两种可能：
# 1) 如果 slave-serve-stale-data 设置为 "yes" (默认值)，slave会继续响应客户端请求，可能是正常数据，也可能是还没获得值的空数据。
# 2) 如果 slave-serve-stale-data 设置为 "no"，slave会回复"正在从master同步（SYNC with master in progress）"来处理各种请求，除了 INFO 和 SLAVEOF 命令。
slave-serve-stale-data yes

# 你可以配置salve实例是否接受写操作。可写的slave实例可能对存储临时数据比较有用(因为写入salve# 的数据在同master同步之后将很容被删除)，但是如果客户端由于配置错误在写入时也可能产生一些问题。
# 从Redis2.6默认所有的slave为只读
# 注意:只读的slave不是为了暴露给互联网上不可信的客户端而设计的。它只是一个防止实例误用的保护层。
# 一个只读的slave支持所有的管理命令比如config,debug等。为了限制你可以用'rename-command'来隐藏所有的管理和危险命令来增强只读slave的安全性。
slave-read-only yes

# 同步策略: 磁盘或socket，默认磁盘方式
repl-diskless-sync no

# 如果非磁盘同步方式开启，可以配置同步延迟时间，以等待master产生子进程通过socket传输RDB数据给slave。
# 默认值为5秒，设置为0秒则每次传输无延迟。
repl-diskless-sync-delay 5

# slave根据指定的时间间隔向master发送ping请求。默认10秒。
# repl-ping-slave-period 10

# 同步的超时时间
# 1）slave在与master SYNC期间有大量数据传输，造成超时
# 2）在slave角度，master超时，包括数据、ping等
# 3）在master角度，slave超时，当master发送REPLCONF ACK pings# 确保这个值大于指定的repl-ping-slave-period，否则在主从间流量不高时每次都会检测到超时
# repl-timeout 60

# 是否在slave套接字发送SYNC之后禁用 TCP_NODELAY
# 如果选择yes，Redis将使用更少的TCP包和带宽来向slaves发送数据。但是这将使数据传输到slave上有延迟，Linux内核的默认配置会达到40毫秒。
# 如果选择no，数据传输到salve的延迟将会减少但要使用更多的带宽。
# 默认我们会为低延迟做优化，但高流量情况或主从之间的跳数过多时，可以设置为“yes”。
repl-disable-tcp-nodelay no

# 设置数据备份的backlog大小。backlog是一个slave在一段时间内断开连接时记录salve数据的缓冲，所以一个slave在重新连接时，不必要全量的同步，而是一个增量同步就足够了，将在断开连接的这段# 时间内把slave丢失的部分数据传送给它。
# 同步的backlog越大，slave能够进行增量同步并且允许断开连接的时间就越长。
# backlog只分配一次并且至少需要一个slave连接。
# repl-backlog-size 1mb

# 当master在一段时间内不再与任何slave连接，backlog将会释放。以下选项配置了从最后一个
# slave断开开始计时多少秒后，backlog缓冲将会释放。
# 0表示永不释放backlog
# repl-backlog-ttl 3600

# slave的优先级是一个整数展示在Redis的Info输出中。如果master不再正常工作了，sentinel将用它来选择一个slave提升为master。
# 优先级数字小的salve会优先考虑提升为master，所以例如有三个slave优先级分别为10，100，25，sentinel将挑选优先级最小数字为10的slave。
# 0作为一个特殊的优先级，标识这个slave不能作为master，所以一个优先级为0的slave永远不会被# sentinel挑选提升为master。
# 默认优先级为100
slave-priority 100

# 如果master少于N个延时小于等于M秒的已连接slave，就可以停止接收写操作。
# N个slave需要是“oneline”状态。
# 延时是以秒为单位，并且必须小于等于指定值，是从最后一个从slave接收到的ping（通常每秒发送）开始计数。
# 该选项不保证N个slave正确同步写操作，但是限制数据丢失的窗口期。
# 例如至少需要3个延时小于等于10秒的slave用下面的指令：
# min-slaves-to-write 3
# min-slaves-max-lag 10

# 两者之一设置为0将禁用这个功能。
# 默认 min-slaves-to-write 值是0（该功能禁用）并且 min-slaves-max-lag 值是10。

######################### 安全 #########################

# 要求客户端在处理任何命令时都要验证身份和密码。
# requirepass foobared

# 命令重命名
# 在共享环境下，可以为危险命令改变名字。比如，你可以为 CONFIG 改个其他不太容易猜到的名字，这样内部的工具仍然可以使用。
# 例如：
# rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
# 也可以通过改名为空字符串来完全禁用一个命令
# rename-command CONFIG ""
# 请注意：改变命令名字被记录到AOF文件或被传送到从服务器可能产生问题。

######################### 限制 客户端#########################

# 设置最多同时连接的客户端数量。默认这个限制是10000个客户端，然而如果Redis服务器不能配置
# 处理文件的限制数来满足指定的值，那么最大的客户端连接数就被设置成当前文件限制数减32（因为Redis服务器保留了一些文件描述符作为内部使用）
# 一旦达到这个限制，Redis会关闭所有新连接并发送错误'max number of clients reached'
# maxclients 10000

# 不要使用比设置的上限更多的内存。一旦内存使用达到上限，Redis会根据选定的回收策略（参见：maxmemmory-policy）删除key。
# 如果因为删除策略Redis无法删除key，或者策略设置为 "noeviction"，Redis会回复需要更多内存的错误信息给命令。例如，SET,LPUSH等等，但是会继续响应像Get这样的只读命令。
# 在使用Redis作为LRU缓存，或者为实例设置了硬性内存限制的时候（使用 "noeviction" 策略）
的时候，这个选项通常事很有用的。
# 警告：当有多个slave连上达到内存上限时，master为同步slave的输出缓冲区所需内存不计算在使用内存中。这样当移除key时，就不会因网络问题 / 重新同步事件触发移除key的循环，反过来slaves的输出缓冲区充满了key被移除的DEL命令，这将触发删除更多的key，直到这个数据库完全被清空为止。
# 总之，如果你需要附加多个slave，建议你设置一个稍小maxmemory限制，这样系统就会有空闲的内存作为slave的输出缓存区(但是如果最大内存策略设置为"noeviction"的话就没必要了)
# maxmemory <bytes>

# 最大内存策略：如果达到内存限制了，Redis如何选择删除key。
# volatile-lru -> 根据LRU算法删除设置过期时间的key
# allkeys-lru -> 根据LRU算法删除任何key
# volatile-random -> 随机移除设置过过期时间的key
# allkeys-random -> 随机移除任何key
# volatile-ttl -> 移除即将过期的key(minor TTL)
# noeviction -> 不移除任何key，只返回一个写错误
# 注意：对所有策略来说，如果Redis找不到合适的可以删除的key都会在写操作时返回一个错误。# 目前为止涉及的命令：set setnx setex append incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby zunionstore zinterstore hset hsetnx hmset hincrby incrby decrby getset mset msetnx exec sort
# 默认策略:
# maxmemory-policy noeviction

# LRU和最小TTL算法的实现都不是很精确，但是很接近（为了省内存），所以你可以用样本量做检测。 例如：默认Redis会检查3个key然后取最旧的那个，你可以通过下面的配置指令来设置样本的个数。
# 默认值为5，数字越大结果越精确但是会消耗更多CPU。
# maxmemory-samples 5

######################### APPEND ONLY MODE ######################### aof

# 默认情况下，Redis是异步的把数据导出到磁盘上。这种模式在很多应用里已经足够好，但Redis进程出问题或断电时可能造成一段时间的写操作丢失(这取决于配置的save指令)。
# AOF是一种提供了更可靠的替代持久化模式，例如使用默认的数据写入文件策略（参见后面的配置）。
# 在遇到像服务器断电或单写情况下Redis自身进程出问题但操作系统仍正常运行等突发事件时，Redis能只丢失1秒的写操作。
# AOF和RDB持久化能同时启动并且不会有问题。
# 如果AOF开启，那么在启动时Redis将加载AOF文件，它更能保证数据的可靠性。
appendonly no

# AOF文件名（默认："appendonly.aof"）
appendfilename "appendonly.aof"

# fsync() 系统调用告诉操作系统把数据写到磁盘上，而不是等更多的数据进入输出缓冲区。
# 有些操作系统会真的把数据马上刷到磁盘上；有些则会尽快去尝试这么做。
# Redis支持三种不同的模式：
# no：不要立刻刷，只有在操作系统需要刷的时候再刷。比较快。
# always：每次写操作都立刻写入到aof文件。慢，但是最安全。
# everysec：每秒写一次。折中方案。
# 默认的 "everysec" 通常来说能在速度和数据安全性之间取得比较好的平衡。
# appendfsync always
appendfsync everysec
# appendfsync no

# 如果AOF的同步策略设置成 "always" 或者 "everysec"，并且后台的存储进程（后台存储或写入AOF 日志）会产生很多磁盘I/O开销。某些Linux的配置下会使Redis因为 fsync()系统调用而阻塞很久。
# 注意，目前对这个情况还没有完美修正，甚至不同线程的 fsync() 会阻塞我们同步的write(2)调用。
# 为了缓解这个问题，可以用下面这个选项。它可以在 BGSAVE 或 BGREWRITEAOF 处理时阻止fsync()。
# 这就意味着如果有子进程在进行保存操作，那么Redis就处于"不可同步"的状态。
# 这实际上是说，在最差的情况下可能会丢掉30秒钟的日志数据。（默认Linux设定）
# 如果把这个设置成"yes"带来了延迟问题，就保持"no"，这是保存持久数据的最安全的方式。
no-appendfsync-on-rewrite no

# 自动重写AOF文件。如果AOF日志文件增大到指定百分比，Redis能够通过 BGREWRITEAOF 自动重写AOF日志文件。# 工作原理：Redis记住上次重写时AOF文件的大小（如果重启后还没有写操作，就直接用启动时的AOF大小）
# 这个基准大小和当前大小做比较。如果当前大小超过指定比例，就会触发重写操作。你还需要指定被重写日志的最小尺寸，这样避免了达到指定百分比但尺寸仍然很小的情况还要重写。
# 指定百分比为0会禁用AOF自动重写特性。
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

# 如果设置为yes，如果一个因异常被截断的AOF文件被redis启动时加载进内存，redis将会发送日志通知用户。如果设置为no，erdis将会拒绝启动。此时需要用"redis-check-aof"工具修复文件。
aof-load-truncated yes

######################### 集群 

# 只有开启了以下选项，redis才能成为集群服务的一部分
# cluster-enabled yes

# 配置redis自动生成的集群配置文件名。确保同一系统中运行的各redis实例该配置文件不要重名。
# cluster-config-file nodes-6379.conf

# 集群节点超时毫秒数。超时的节点将被视为不可用状态。
# cluster-node-timeout 15000

# 如果数据太旧，集群中的不可用master的slave节点会避免成为备用master。如果slave和master失联时间超过:
 (node-timeout * slave-validity-factor) + repl-ping-slave-period
则不会被提升为master。
# 如node-timeout为30秒，slave-validity-factor为10, 默认default repl-ping-slave-period为10秒,失联时间超过310秒slave就不会成为master。
# 较大的slave-validity-factor值可能允许包含过旧数据的slave成为master，同时较小的值可能会阻止集群选举出新master。
#为了达到最大限度的高可用性，可以设置为0，即slave不管和master失联多久都可以提升为master
# cluster-slave-validity-factor 10

# 只有在之前master有其它指定数量的工作状态下的slave节点时，slave节点才能提升为master。默认为1（即该集群至少有3个节点，1 master＋2 slaves，master宕机，仍有另外1个slave的情况下其中1个slave可以提升）
# 测试环境可设置为0，生成环境中至少设置为1
# cluster-migration-barrier 1

# 默认情况下如果redis集群如果检测到至少有1个hash slot不可用，集群将停止查询数据。
# 如果所有slot恢复则集群自动恢复。
# 如果需要集群部分可用情况下仍可提供查询服务，设置为no。
# cluster-require-full-coverage yes

######################### 慢查询日志 #########################

# 慢查询日志，记录超过多少微秒的查询命令。查询的执行时间不包括客户端的IO执行和网络通信时间，只是查询命令执行时间。
# 1000000等于1秒，设置为0则记录所有命令
slowlog-log-slower-than 10000

# 记录大小，可通过SLOWLOG RESET命令重置
slowlog-max-len 128

```

### 11 Redis 持久化

#### 1 RDB

生产环境备份rdb

![图片来源简书](https://upload-images.jianshu.io/upload_images/10939682-807dfdd5195f9d20.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)



> 在指定的时间间隔中,将内存中的数据集快照写入磁盘,恢复的时候,将快照文件直接读取到内存中
>
> Redis会单独创建(fork)一个子进程来进行持久化,会线将数据写入到一个临时文件中,待持久化过程都结束了,再用这个临时文件替换上次持久化好的文件,整个过程中,主进程是不做任何IO操作的,这就确保了极高的性能,如果需要进行大规模的数据恢复,且对于数据恢复的完整性不是非常敏感,那RDB的方式要比AOF的方式更加高效,RDB的缺点是最后一次持久化数据可能会丢失

**触发机制**

* save 的规则满足的情况下,自动出发rdb规则
* 执行flushall 命令,也会出发rdb规则
* 退出redis也会生成rdb文件

**恢复rdb文件**

* 只要将rdb文件放在redis的启动目录就可以,redis启动的时候会自动检查dump.rdb恢复其中的数据

```bash
# 查看配文件的位置
config get dir
# 如果在dir目录下存在rdb文件,那么在启动的时候会自动恢复其中的数据
```

**优点:**

1. 适合大规模数据恢复
2. 如果对数据的完整性要求不高

**缺点:**

1. 需要一定的时间间隔进程操作,如果redis宕机了,这个最后一次修改的数据就没了

2. fork进程的时候,会占用一定的内容空间

   

#### 2 AOF(Append Only File)

将我们的所有命令都记录下来,history,恢复的时候就把这个文件全部执行一遍

![image-20201227183918572](image-20201227183918572.png)

> 以日志的形式记录每个写操作,将Redis执行过的命令记录写来(读操作不记录)，只许追加文件但不可以改写文件，redis启动之初会读取该文件重新构建数据，换言之，redis重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作

`AOF保存的是 appendonly.aof`

默认不开启，需要手动配置，将配置文件appendonly改为yes就开启了aof

如果这个apf文件有错位，redis会启动失败，我们需要修复这个文件

redis给我们体用了一个工具 `redis-check-aof --fix appendonly.aof `

如果文件正常，重启就可以恢复

> 重写规则，如果AOF文件大于64m，太大了，fork一个新的进程来进行文件的重写

**优点：**

1. 每一次修改都同步，文件的完整性会更好
2. 每秒同步一次，可能会丢失一秒的数据
3. 从不同步，效率是最高的

**缺点：**

1. 相对于数据文件来说，aof远远大于rdb，修复速度也比rdb慢
2. Aof运行效率也要比rdb慢，所以我们redis默认的配置就是rdb持久化



### 12 Redis订阅发布

![image-20201227201729862](image-20201227201729862.png)

> 发布订阅（pub/sub）是一种消息通信模式，发送者（pub）发送消息，订阅者（sub）接收消息，微博，微信，Redis客户端可以订阅任意数量的频道

![image-20201227202915634](image-20201227202915634.png)

```bash
# 订阅
127.0.0.1:6379> subscribe 1
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "1" # 消息频道
3) (integer) 1
1) "message" 
2) "1"  # 消息频道
3) "111"# 具体内容

# 发布
127.0.0.1:6379> publish 1 111
(integer) 1
```

**应用**

* 聊天室
* 公众号

### 13 Redis集群环境搭建

> 主从复制，是指将一台 Redis服务器的数据，复制到其他的 Redist服务器。前者称为主节点( master/leader),后者称为从节点
>  ( slave/ allowed);数据的复制是单向的，只能由主节点到从节点， Master以写为主， Slave以读为主
>  默认情况下，每台 Redis服务器都是主节点：且一个主节点可以有多个从节点（或没有从节点，但一个从节点只能有一个主节点。

**主从复制的作用**

* 数据冗余: 主从复制实现了数据的热备份,是持久化之外的一种冗余方式
* 故障恢复: 当主节点出现问题时,可以由从节点提供服务,实现快速的故障恢复,实际上是一种服务冗余
* 负载均衡: 在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务（即写 Redis数据时应用连接主节点，读 Redis数时应用连接从节点），分担服务器负載：尤其是在写少读多的场原下，通过多个从节点分担读负载，可以大大提高 Redish服务器的并发量
* 高可用基石：除了上述作用以外，主从复制还是哨兵和集群能够实施的基础，因此说主从复制是 Redist高可用的基础

总的来说，要将 Redis运用于工程项目中，只使用一台 Redis!是万万不能的，原因如下
 1、从结构上，单个 Redis服务器会发生单点故障。，井且一台服务器需要处理所有的请求负载，压力较大
 2、从容量上，单个 Redis:服务器内存容量有限，就算一台 Redis:服务器内存容量为256G,也不能将所有内存用作 Redis存储内存

一般来说，单台 Redist最大使用内存不应该超过20G电商网站上的商品，一般都是一次上传，无数次浏览的，说专业点也就是”多读少写**

#### 1 环境配置

只配置从库,无需配置主库

```bash
# 查看当前库的信息
127.0.0.1:6379> info replication 
# Replication
role:master # 角色
connected_slaves:0 # 没有从机
master_replid:27a8d6b5022631dd44b5eb22816560dcd2b2fc68
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0

#########################
# 命令方式
#########################
# 默认启动三台机器每一台机器都是一台主机
# 从机输入命令 SLAVEOF ip port 认定主机

#########################
# 配置文件
#########################
replicaof <ip> <port>
```

> 测试：主机断开连接，从机依旧连接到主机的，但是没有写操作，这个时候，主机如果回来了，从机依旧可以直接获取到主机写的
>  信息！
>  如果是使用命令行来配置的主从，这个时如果重启了，就会变回**主机！只要变为从机，立马就会从主机中获取值**

**复制原理**
 		Slave启动成功连接到 master 后会发送一个sync同步命令，Master接到命令，启动后台的存盘进程，同时枚集所有接收到的用于修改数据集命令，在后台进程执行完毕之后， master将传送整个数据文件到ave,并完成一次完全同步
**全量复制**：

​		而 slave服务在接收到数据库文件数后，将其存盘井加载到内存中
**增量复制：**

​		 Master继续将新的所有收集到的修改命令依次传给slave,完成同步
 但是只要是重新连接 master,一次完全同步（全量复制）将被自动执行

#### 2 宕机后手动配置主机

执行命令

```bash
slaveof no one # 从机变成主机命令（即便是主机回来了，变成主机的从机，仍然是主机）
```

#### 3 哨兵模式

> 当主机挂掉之后，自动推选出一个主机

> 主从切换技术的方法是：当主服务器宕机后，需要手动把一台从服务器切換为主服务器，这就需要人工千预，费事费力，还会造成段时间内服务不可用。这不是一种推荐的方式，更多时候，我们优先考虑哨兵模式。
>
>  Redis从2.8开始正式提供了 Sentinel(哨兵）架构来解决这个问题
> 能够后台监控主机是否故障，如果故了根据投票数自动将从库转换为主库。
>
> 消兵模式是一种特殊的模式，首先 Redis提供了哨兵的命令，哨兵是一个独立的进程，作为进程，它会独立运行，其原理是哨兵通
>  过发送命令，等待 Redis!服务器响应，从而监控运行的多个 Redis实例

![image-20201227211940620](image-20201227211940620.png)

![image-20201227212001656](image-20201227212001656.png)

**这里的哨兵有两个作用**
 通过发送命令，让Redis服务器返回监控其运行状态，包括主服务器和从服务器。当哨兵监测到 master宕机，会自动将 slave切換成 master,然后通过发布订阅模式通知其他的从服务器，修改配置文件，让它们切换主机。
 然而一个哨兵进程对 Redish服务器进行监控，可能会出现问题，为 此，我们可以使用多个哨兵进行监控。各个哨兵之间还会进行监
 控，这样就形成了多哨兵模式。

```bash
vim sentinel.conf
-------------
# 配置哨兵配置文件
# sentinel monitor <名称>  <ip> <port>  <主机挂机，从机投票，看谁来接替主机，票数最多的成为主机>
sentinel nonitor myredis 127.0.0.1 6379 1
-------------
# 启动哨兵
redis-sentinel sentinel.conf
```

如果主机恢复了，只能归并到新的主机下，当做从机，这就是哨兵模式的规则

**优点：**

1. 哨兵集群，基于主从复制模式，具备所有的主从配置优点

2. 主从可以切换，故障可以转移，系统的可用性为编号
3. 哨兵模式就是主从赋值的升级，手动到自动，更加健壮

**缺点**：

1. Redis不好在线扩容，集群数量一旦达到上限，在线扩容比较困难
2. 实现哨兵模式的配置很麻烦，有很多选择

### 14 缓存穿透与缓存雪崩

​		Redis缓存的使用，极大的提升了应用程序的性能和效率。，特别是数据查询方面。但同时，它也带来了一些问题。其中，最要害的
 问题，就是数据的一致性问题，从严格意义上讲，这个问题无解。如果对数据的一致性要求很高，那么就不能使用缓， 另外的一些典型问题就是，缓存穿透、缓存雪和线存击穿。目前，业界也都有比较流行的解决方案。

#### 1 缓存穿透

![image-20201227213614378](image-20201227213614378.png)

缓存穿透的概念很简单，用户想要查询一个数据，发现 redis内存数据库没有，也就是缓存没有命中，于是向持久层数据库查询。发现也没有，于是本次直询失败，当用户很多的时候，缓存都没有命中，于是都去请求了持久层数据库，这会给持久层数据库造成很大的压力，这时候就相当于出现了缓存穿透。

##### 1 布过滤器

布过滤器是一种数据结构，对所有可能查询的参数以hash形式存储，在控制层先进行校验，不符合则丢弃，从而避免了对底层存
储系统的查询压力

![image-20201227213657749](image-20201227213657749.png)

##### 2  缓存空对象

当存储层不命中后，即使返回的空对象也将具缓存起来，同时会设置一个过期时间，之后再访问这个数据将会从缓存中获取，保护了后端数据源。

![image-20201227213813078](image-20201227213813078.png)

但是这种方法会存在两个问题
 1、如果空值能够被存起来，这就意味着缓存要更多的空间存储更多的键，因为这当中可能会有很多的空值的健
 2、即使对空值设置了过期时间，还是会存在援存层和存储层的数据会有一段时间园口的不一致，这对于需要保持一致性的业务会有影响

#### 2 缓存击穿

这里需要注意和缓存穿透的区別，缓存击穿，是指一个key非常热点，在不停的扛着大井发，大井发集中对这一个点进行访问，当 这个key在失效的瞬间，持续的大井发就穿破缓存，直接请求数起库，就像在一个屏障上凿开了一个洞。

当某个key在过期的瞬间，有大量的请求并发访问，这类数据一般是热点数据，由于缓存过期，会同时访问数据库来查询最新数据，井且回写缓存，会导致数据库瞬间压力过大

##### 1 设置热点数据永不过期

从缓存层面来看，没有设置过期时间，所以不会出现key过期后产生的问题

##### 2 加互斥锁

分布式锁：使用分布式锁，保证对于每个key同时只有一个线程去查询后服务，其他线程没有获得分布式的权限，因此只需要等待即可。这种方式将高并发的压力转移到了分布式，因此对分布式模的考验很大。

#### 3 缓存雪崩

缓存雪崩，是指在某一个时间段，缓存集中过期失效， Redis宕机

产生雪崩的原因之一，比如在写本文的时候，马上就要到双十二零点，很快就会迎来一波抢购，这波商品时间比较集中的放入了缓存，假设援存一个小时。那么到了凌晨一点钟的时候，这批商品的援存就都过期了。而对这批商品的访问查询，都落到了数据库上，对于数据库而言，就会产生周期性的压力波峰。于是所有的请求都会达到存储层，存储层的调用量会暴増，造成存储层也会挂掉的情况



##### 1 Redis高可用

这个思想的含义是，既然 tredis?有可能挂掉，那我多增设几台 redis,这样一台挂掉之后其他的还可以继续工作，其实就是搭建的集群(异地多活！)

##### 2 限流降级

这个解决方案的思想是，在缓存失效后，通过加或者队列来控制读数据库写缓存的线程数量，比如对某个key只允许一个线程查询数据和写缓存，其他线程等待

##### 3 数据预热

数据加热的含义就是在正式部署之前，我先把可能的数据先预先访问一遍，这样部分可能大量访问的数据就会加载到缓存中。在即将发生大井发访问前手动触发加加载缓存存不同的key,设置不同的过期时间，让缓存失效的时间点，尽量均匀。