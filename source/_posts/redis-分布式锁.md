---
title: redis 分布式锁
top: false
cover: false
toc: true
mathjax: true
categories:
  - 中间件
  - redis
  - 分布式锁 
tags:
  - 中间件
  - redis
  - 分布式锁
date: 2021-07-31 17:20:21
password:
summary:
---

## 分布式锁篇

## 一、引言

​		我们在系统中修改已有数据时，需要先读取，然后进行修改保存，此时很容易遇到并发问题。由于修改和保存不是原子操作，在并发场景下，部分对数据的操作可能会丢失。在单服务器系统我们常用本地锁来避免并发带来的问题，然而，当服务采用集群方式部署时，本地锁无法在多个服务器之间生效，这时候保证数据的一致性就需要分布式锁来实现。

![img](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210731180556.png)

## 二、概述

​		首先，锁在传统多线程环境下，即保证多个线程取操作共享资源的安全性。扩展到分布式的环境中，由于服务是多节点部署，且网络因素也会有一定的影响，因此要保证分布式环境共享资源的正确性，一致性，因此用的分布式锁。分布式锁提供了多个服务器节点访问共享资源互斥的一种手段。

### 2.1 常见的分布式锁

* 基于数据库
* 基于缓存
* 基于zookeeper实现的分布式锁。

### 2.2 几个重要的概念

- 分布式锁管理(DLM): Distributed Lock Manager，分配给节点锁和释放锁，对分布式锁进行统一的管理
- 租约(lease): 从DLM, 获得锁的有效期，超过有效期，DLM自动回收锁，租约保证了一个客户端不会占用锁很长时间。一定程度上保证了高效性。

### 2.3 分布式锁的特点

- 互斥性：和我们本地锁一样互斥性是最基本，但是分布式锁需要保证在不同节点的不同线程的互斥。
- 可重入性：同一个节点上的同一个线程如果获取了锁之后那么也可以再次获取这个锁。
- 锁超时：和本地锁一样支持锁超时，防止死锁。
- 高效，高可用：加锁和解锁需要高效，同时也需要保证高可用防止分布式锁失效，可以增加降级。
- 支持阻塞和非阻塞：和ReentrantLock一样支持lock和trylock以及tryLock(long timeOut)。
- 支持公平锁和非公平锁(可选)：公平锁的意思是按照请求加锁的顺序获得锁，非公平锁就相反是无序的。这个一般来说实现的比较少。

## 三、如何设计分布式锁

### 3.1 基于数据库

#### 3.1.1 数据表方案

要实现分布式锁，最简单的方式可能就是直接创建一张锁表，然后通过操作该表中的数据来实现了。

当我们要锁住某个方法或资源时，我们就在该表中增加一条记录，想要释放锁的时候就删除这条记录。

创建表：

```sql
create table TDistributedLock (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `lock_key` varchar(64) NOT NULL DEFAULT '' COMMENT '锁的键值',
  `lock_timeout` datetime NOT NULL DEFAULT NOW() COMMENT '锁的超时时间',
  `create_time` datetime NOT NULL DEFAULT NOW() COMMENT '记录创建时间',
  `modify_time` datetime NOT NULL DEFAULT NOW() COMMENT '记录修改时间',
  PRIMARY KEY(`id`),
  UNIQUE KEY `uidx_lock_key` (`lock_key`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='分布式锁表';
```

需要加锁时，只需要往这张表里插入一条数据就可以了：

```sql
INSERT INTO TDistributedLock(lock_key, lock_timeout) values('lock_xxxx', '2020-07-19 18:20:00');
```

当对共享资源的操作完毕后，可以释放锁：

```sql
DELETE FROM TDistributedLock where lock_key='lock_xxxx';
```

该方案简单方便，主要利用数据库表的唯一索引约束特性，保证多个微服务模块同时申请分布式锁时，只有一个能够获得锁。

下面讨论下一些相关问题：

1. 获得锁的微服务模块意外crash，来不及释放锁怎么办？因为在插入锁记录时，同时设置了一个`lock_timeout`属性，可以另外跑一个`lock_cleaner`，将超时的锁记录删除。当然为了安全，`lock_timeout`最好设置一个合理的值，以确保在这之后正常的共享资源操作一定是完成了的。
2. 获得锁失败的微服务模块如何继续尝试获得锁？搞一个while循环，反复尝试，如能成功获得锁就跳出循环，否则sleep一会儿重新进入循环体继续尝试。
3. 数据库单点不安全怎么办？数据库领域有主从、一主多从、多主多从等复制方案，可保证一个数据库实例挂掉时，其它实例可以接管过来继续提供服务。不过需要注意的是有一些复制方案它是异步的，并不能保证写入一个数据库实例的数据立马可以在另外一个数据库实例中查询到， 这就容易造成锁丢失，导致授予了两把锁的问题。
4. 获得锁的微服务模块想重入地获得锁怎么办？在数据库表中加个字段，记录当前获得锁的机器的主机信息和线程信息，那么下次再获取锁的时候先查询数据库，如果当前机器的主机信息和线程信息在数据库可以查到的话，直接把锁分配给它就可以了。同样为了确保安全，必须在本微服务模块没有主动释放锁，同时离锁的超时时间还很久远的情况下，才可以安全地直接把锁再次分配给它，同时更新锁的超时时间。

该方案的缺陷：

1. 该方案利用数据库表自身的唯一键约束，性能相比后面提到的redis方案会差一点。
2. 该方案中其它尝试获得锁的客户端会反复尝试插入数据，消耗数据库资源。
3. 该方案中需要为数据库找一种比较可靠的高可用方案，同时还得确保数据库实例之间复制方案是满足一致性要求的。

#### 3.1.2 数据库表的行排它锁方案

除了动态地往数据库表里插入数据外，还可以预先将锁信息写入数据库表，然后利用数据库的行排它锁来进行加锁与释放锁操作。

例如加锁时可以执行以后命令：

```sql
SET autocommit = 0;
START TRANSACTION;
SELECT * FROM TDistributedLock WHERE lock_key='lock_xxxx' FOR UPDATE;
```

当对共享资源的操作完毕后，可以释放锁：

```sql
COMMIT;
```

该方案也很简单，利用数据库表的行排它锁特性，保证多个微服务模块同时申请分布式锁时，只有一个能够获得锁。

下面讨论下一些相关问题：

1. 获得锁的微服务模块意外crash，来不及释放锁怎么办？没有`COMMIT`，数据库会自动将锁释放掉。
2. 获得锁失败的微服务模块如何继续尝试获得锁？`for update`语句会在执行成功后立即返回，在执行失败时一直处于阻塞状态，直到成功，因此不用写while循环反复尝试获得锁了。
3. 数据库单点问题跟上面那个方案一致，不再赘述。
4. 在这个方案下，好像很难做到锁可重入了。

该方案的缺陷：

1. 该方案利用数据库表的行排它锁特性，性能上比上面那个方案好一点，但相比后面提到的redis方案还是差一点。
2. 该方案中需要为该表的数据库事务配置较高的事务超时时间，而且一个排他锁长时间不提交，会占用数据库连接。
3. 该方案中同样需要为数据库找一种比较可靠的高可用方案，同时还得确保数据库实例之间复制方案是满足一致性要求的。

### 3.2 基于redis

使用redis做分布式锁的思路大概是这样的：在redis中设置一个值表示加了锁，然后释放锁的时候就把这个key删除。

Redis 锁主要利用 Redis 的 setnx 命令。

- 加锁命令：SETNX key value，当键不存在时，对键进行设置操作并返回成功，否则返回失败。KEY 是锁的唯一标识，一般按业务来决定命名。
- 解锁命令：DEL key，通过删除键值对释放锁，以便其他线程可以通过 SETNX 命令来获取锁。
- 锁超时：EXPIRE key timeout, 设置 key 的超时时间，以保证即使锁没有被显式释放，锁也可以在一定时间后自动释放，避免资源被永远锁住。

则加锁解锁伪代码如下：

```ruby
if (setnx(key, 1) == 1){
    expire(key, 30)
    try {
        //TODO 业务逻辑
    } finally {
        del(key)
    }
}
```

上述锁实现方式存在一些问题：

#### 3.2.1 SETNX 和 EXPIRE 非原子性

如果 SETNX 成功，在设置锁超时时间后，服务器挂掉、重启或网络问题等，导致 EXPIRE 命令没有执行，锁没有设置超时时间变成死锁。

![img](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210731183406.png)

有很多开源代码来解决这个问题，比如使用 lua 脚本。

具体代码如下：

**获取锁**

```lua
-- NX是指如果key不存在就成功，key存在返回false，PX可以指定过期时间
redis.call("set", key, ARGV[1], "NX", "PX", ARGV[2])
```

**释放锁**

```lua
-- 释放锁涉及到两条指令，这两条指令不是原子性的
-- 需要用到redis的lua脚本支持特性，redis执行lua脚本是原子性的
if redis.call("get", key) == ARGV[1] then
	return redis.call("del", key)
else
	return 0
end
```

上述的lua代码比较简单，不具体解释了。这里要注意给锁键的value值要保证唯一，这个是为了避免释放错了锁。

**场景如下：**

​		假设A获取了锁，过期时间30s，此时35s之后，锁已经自动释放了，A去释放锁，但是此时可能B获取了锁。加上释放锁前的value值判断，A客户端就不能删除B的锁了。

如果是javascript的项目，可以使用[redislock](https://github.com/danielstjules/redislock)库，它封装了上述加锁、释放锁等操作逻辑，用起来很方便。

才区区几行代码就优雅地搞定了分布式锁，而且redis的写入、删除键值的性能超高，看样子很完美，但事实上并非如此。

#### 3.2.2 锁误解除

​		如果线程 A 成功获取到了锁，并且设置了过期时间 30 秒，但线程 A 执行时间超过了 30 秒，锁过期自动释放，此时线程 B 获取到了锁；随后 A 执行完成，线程 A 使用 DEL 命令来释放锁，但此时线程 B 加的锁还没有执行完成，线程 A 实际释放的线程 B 加的锁。

![img](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210731184253.png)

通过在 value 中设置当前线程加锁的标识，在删除之前验证 key 对应的 value 判断锁是否是当前线程持有。可生成一个 UUID 标识当前线程，使用 lua 脚本做验证标识和解锁操作。

**获取锁**

```lua
-- 获取UUID
String uuid = UUID.randomUUID().toString().replaceAll("-","");
SET key uuid NX EX 30
redis.call("set", key, ARGV[1], "NX", "PX", ARGV[2])
```



```ruby

// 解锁
if (redis.call('get', KEYS[1]) == ARGV[1])then 
    return redis.call('del', KEYS[1])
else
    return 0
end
```

#### 3.2.3 超时解锁导致并发

如果线程 A 成功获取锁并设置过期时间 30 秒，但线程 A 执行时间超过了 30 秒，锁过期自动释放，此时线程 B 获取到了锁，线程 A 和线程 B 并发执行。

![img](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210731233037.png)

A、B 两个线程发生并发显然是不被允许的，一般有两种方式解决该问题：

- 将过期时间设置足够长，确保代码逻辑在锁释放之前能够执行完成。
- 为获取锁的线程增加守护线程，为将要过期但未释放的锁增加有效时间。

![img](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210731212922.png)

#### 3.2.4 不可重入

当线程在持有锁的情况下再次请求加锁，如果一个锁支持一个线程多次加锁，那么这个锁就是可重入的。如果一个不可重入锁被再次加锁，由于该锁已经被持有，再次加锁会失败。Redis 可通过对锁进行重入计数，加锁时加 1，解锁时减 1，当计数归 0 时释放锁。

在本地记录记录重入次数，如 Java 中使用 ThreadLocal 进行重入次数统计，简单示例代码：

```java
private static ThreadLocal<Map<String, Integer>> LOCKERS = ThreadLocal.withInitial(HashMap::new);
// 加锁
public boolean lock(String key) {
  Map<String, Integer> lockers = LOCKERS.get();
  if (lockers.containsKey(key)) {
    lockers.put(key, lockers.get(key) + 1);
    return true;
  } else {
    if (SET key uuid NX EX 30) {
      lockers.put(key, 1);
      return true;
    }
  }
  return false;
}
// 解锁
public void unlock(String key) {
  Map<String, Integer> lockers = LOCKERS.get();
  if (lockers.getOrDefault(key, 0) <= 1) {
    lockers.remove(key);
    DEL key
  } else {
    lockers.put(key, lockers.get(key) - 1);
  }
}
```

本地记录重入次数虽然高效，但如果考虑到过期时间和本地、Redis 一致性的问题，就会增加代码的复杂性。另一种方式是 Redis Map 数据结构来实现分布式锁，既存锁的标识也对重入次数进行计数。Redission 加锁示例：

```java
// 如果 lock_key 不存在
if (redis.call('exists', KEYS[1]) == 0)
then
    // 设置 lock_key 线程标识 1 进行加锁
    redis.call('hset', KEYS[1], ARGV[2], 1);
    // 设置过期时间
    redis.call('pexpire', KEYS[1], ARGV[1]);
    return nil;
    end;
// 如果 lock_key 存在且线程标识是当前欲加锁的线程标识
if (redis.call('hexists', KEYS[1], ARGV[2]) == 1)
    // 自增
    then redis.call('hincrby', KEYS[1], ARGV[2], 1);
    // 重置过期时间
    redis.call('pexpire', KEYS[1], ARGV[1]);
    return nil;
    end;
// 如果加锁失败，返回锁剩余时间
return redis.call('pttl', KEYS[1]);
```

#### 3.2.5 无法等待锁释放

上述命令执行都是立即返回的，如果客户端可以等待锁释放就无法使用。

- 可以通过客户端轮询的方式解决该问题，当未获取到锁时，等待一段时间重新获取锁，直到成功获取锁或等待超时。这种方式比较消耗服务器资源，当并发量比较大时，会影响服务器的效率。
- 另一种方式是使用 Redis 的发布订阅功能，当获取锁失败时，订阅锁释放消息，获取锁成功后释放时，发送锁释放消息。如下：

![img](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210731214029.png)

#### 3.2.6 Redis高可用架构之主备切换

为了保证 Redis 的可用性，一般采用主从方式部署。主从数据同步有异步和同步两种方式，Redis 将指令记录在本地内存 buffer 中，然后异步将 buffer 中的指令同步到从节点，从节点一边执行同步的指令流来达到和主节点一致的状态，一边向主节点反馈同步情况。

在包含主从模式的集群部署方式中，当主节点挂掉时，从节点会取而代之，但客户端无明显感知。当客户端 A 成功加锁，指令还未同步，此时主节点挂掉，从节点提升为主节点，新的主节点没有锁的数据，当客户端 B 加锁时就会成功。

![img](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210731233048.png)

单点是不行的，主从架构也不保险，对于这个问题，Redis 的官方给出了一个 `RedLock` 的解决方案。

#### 3.2.7 Redis高可用架构之 集群脑裂

集群脑裂指因为网络问题，导致 Redis master 节点跟 slave 节点和 sentinel 集群处于不同的网络分区，因为 sentinel 集群无法感知到 master 的存在，所以将 slave 节点提升为 master 节点，此时存在两个不同的 master 节点。Redis Cluster 集群部署方式同理。

当不同的客户端连接不同的 master 节点时，两个客户端可以同时拥有同一把锁。如下：

![img](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210731233051.png)

#### 3.2.8 RedLock

实现 `RedLock` 实现分布式锁的前提条件是：假设有 N 个 Redis master 节点，所有节点之间是相互独立，互补影响的，并且业务系统也是单纯的对这些节点进行调用：

RedLock主要解决Redis没有总是可用的保障，解决failover问题。加锁的时候，它会向过半节点发送 set(key, value, nx=True, ex=xxx)指令，只要过半节点set成功，那就认为加锁成功。释放锁时，需要向所有节点发送del 指令。不过Redlock算法还需要考虑出错重试、时钟漂移等很多细节问题，同时因为RedLock需要向多个节点进行读写，意味着相比单实例Redis性能会下降一些。

##### 3.2.8.1 具体实现

在Redis分布式环境中，我们假设有N个Redis master。这些节点完全独立，不存在主从复制或者其他集群协调机制。redlock确保在每（N)个实例上使用此方法获取和释放锁。假设有5个不会同时都宕掉的Redis master节点。

为了取到锁，客户端应该执行以下操作:

1. 获取当前Unix时间，以毫秒为单位。
2. 依次尝试从N个实例，使用相同的key和随机值获取锁。在步骤2，当向Redis设置锁时,客户端应该设置一个网络连接和响应超时时间，这个超时时间应该小于锁的失效时间。例如你的锁自动失效时间为10秒，则超时时间应该在5-50毫秒之间。这样可以避免服务器端Redis已经挂掉的情况下，客户端还在死死地等待响应结果。如果服务器端没有在规定时间内响应，客户端应该尽快尝试另外一个Redis实例。
3. 客户端使用当前时间减去开始获取锁时间（步骤1记录的时间）就得到获取锁使用的时间。当且仅当从大多数（这里是3个节点）的Redis节点都取到锁，并且使用的时间小于锁失效时间时，锁才算获取成功。
4. 如果取到了锁，key的真正有效时间等于有效时间减去获取锁所使用的时间（步骤3计算的结果）。
5. 如果因为某些原因，获取锁失败（没有在至少N/2+1个Redis实例取到锁或者取锁时间已经超过了有效时间），客户端应该在所有的Redis实例上进行解锁（即便某些Redis实例根本就没有加锁成功）

##### 3.2.8.2 RedLock缺点

这个方案看似很好，缺点就是太重，通常不被推荐。如果很在乎高可用性，希望挂了一台Redis完全不受影响，那么应该考虑redlock。仔细审视后该方案还是存在一些问题的，分布式架构师`Martin`就提出了自己的[意见](http://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html)。这些意见总结下来如下：

> 分布式锁的用途无非两种：
>
> - 要么为了提升效率，用锁来保证一个任务没有必要被执行两次。比如（很昂贵的计算） 保证正确，使用锁来保证任务按照正常的步骤执行，防止两个节点同时操作一份数据，造成文件冲突，数据丢失。这种情况下对锁是有一定宽容度的，就算发生了两个节点同时工作，对系统的影响也仅仅是多付出了一些计算的成本，没什么额外的影响。这个时候 使用单点的 Redis 就能很好的解决问题，没有必要使用RedLock，维护那么多的Redis实例，提升系统的维护成本。
> - 要么为了安全性，此时必须从理论上确保拿到的锁是安全的，两个进程不可能同时持有某把锁，因为很可能涉及到一些金钱交易，如果锁定失败，并且两个节点同时处理同一数据，则结果将导致文件损坏，数据丢失，永久性不一致，或者金钱方面的损失！而RedLock方案从理论上说并不能保证这一点。

`RedLock`方案从理论上说并不能保证锁的安全性，主要有以下几点原因：

1. 锁的自动过期机制很可能导致锁的意外丢失。因此采用watchdog机制确保在持有锁之后持续地给锁续期是有必要的。
2. `RedLock`方案对于系统时钟有强依赖。假设有A、B、C、D、E 5个redis节点：
   1. 客户端1获取节点A，B，C的锁定。由于网络问题，无法访问D和E。
   2. 节点C上的时钟向前跳，导致锁过期。
   3. 客户端2获取节点C，D，E的锁定。由于网络问题，无法访问A和B。
   4. 现在，客户1和2都认为他们持有该锁。
3. `RedLock`方案同样无法避免redis实例意外重启导致的问题。假设有A、B、C、D、E 5个redis节点：
   1. 客户端1获取节点A，B，C的锁定。由于网络问题，无法访问D和E。
   2. 节点C上的redis实例意外重启，重启后原来写入的键值丢失。
   3. 客户端2获取节点C，D，E的锁定。由于网络问题，无法访问A和B。
   4. 现在，客户1和2都认为他们持有该锁。

虽说`RedLock`从理论上说确实无法100%保证锁的安全性，但以上列举的场景极为严苛，事实上在现实中很难碰到。而由于该方案获取锁的效率确实很高，事实上还是有不少业务场景就是使用的该方案。

##### 3.2.8.3 `Redisson` 工具中的 `RedLock` 实现

`Redisson` 是一个具备内存数据网格特征的 Redis Java 客户端工具。它提供了 30 多种对象类型和服务功能：Set, Multimap, SortedSet, Map, List, Queue, Deque, Semaphore, Lock, AtomicLong, Map Reduce, Publish / Subscribe, Bloom filter, Spring Cache, Tomcat, Scheduler, JCache API, Hibernate, RPC.

Redisson 对于 RedLock 的实现：

```java
Config config = new Config();
config.useSingleServer().setAddress("redis://192.168.120.0:5378").setPassword("123456").setDatabase(0);
RedissonClient client = Redisson.create(config);

Config config1 = new Config();
config1.useSingleServer().setAddress("redis://192.168.120.1:5378").setPassword("123456").setDatabase(0);
RedissonClient client1 = Redisson.create(config1);

Config config2 = new Config();
config2.useSingleServer().setAddress("redis://192.168.120.2:5378").setPassword("123456").setDatabase(0);
RedissonClient client2 = Redisson.create(config2);

String key = "SIGN_GET_GIFT_" + userId;
RLock rLock = client.getLock(key);
RLock rLock1 = client1.getLock(key);
RLock rLock2 = client2.getLock(key);
RedissonRedLock redLock = new RedissonRedLock(rLock, rLock1, rLock2);
try {
    if (!redLock.tryLock()) {
        return "签到失败，请勿频繁请求";
    }
    //执行签到领取礼品逻辑
    signGetGift();
    saveSignLog();
    return "签到成功";
} catch (Exception e) {
    return "签到异常";
} finally {
    redLock.unlock();
}
```



### 3.3 基于zookeeper

`zookeeper`是一种提供配置管理、分布式协同以及命名的中心化服务。很明显`zookeeper`本身就是为了分布式协同这个目标而生的，它采用一种被称为ZAB(Zookeeper Atomic Broadcast)的一致性协议来保证数据的一致性。基于zk的一些特性，我们很容易得出使用zk实现分布式锁的落地方案：

1. 使用zk的临时节点和有序节点，每个线程获取锁就是在zk创建一个临时有序的节点，比如在/lock/目录下。
2. 创建节点成功后，获取/lock目录下的所有临时节点，再判断当前线程创建的节点是否是所有的节点的序号最小的节点
3. 如果当前线程创建的节点是所有节点序号最小的节点，则认为获取锁成功。
4. 如果当前线程创建的节点不是所有节点序号最小的节点，则对节点序号的前一个节点添加一个事件监听。 比如当前线程获取到的节点序号为`/lock/003`,然后所有的节点列表为`[/lock/001,/lock/002,/lock/003]`,则对`/lock/002`这个节点添加一个事件监听器。

如果锁释放了，会唤醒下一个序号的节点，然后重新执行第3步，判断是否自己的节点序号是最小。

比如`/lock/001`释放了，`/lock/002`监听到时间，此时节点集合为`[/lock/002,/lock/003]`,则`/lock/002`为最小序号节点，获取到锁。

从数据一致性的角度来说，zk的方案无疑是最可靠的，而且等候锁的客户端也不用不停地轮循锁是否可用，当锁的状态发生变化时可以自动得到通知。

但zookeeper实现也存在较大的缺陷：

- 性能问题

zookeeper作为分布式协调系统，不适合作为频繁的读写存储系统。而且通过增加zookeeper服务器来提高集群的读写能力也是有上限的，因为zookeeper集群规模越大，zookeeper数据需要同步到更多的服务器。同时zookeeper分布式锁每一次都要申请锁和释放锁，都要动态创建删除临时节点，所以zookeeper不能维护大量的分布式锁，也不能维护大量的客户端心跳长连接。在分布式定理中，zookeeper追求的是CP，也就是zookeeper保证集群向外提供统一的视图，但是zookeeper牺牲了可用性，在极端情况下，zookeeper可能会丢弃一些请求。并且zookeeper集群在进行leader选举的时候整个集群也是不可用的，集群选举时间长达30 ~ 120s。

- 惊群效应

在获取锁的时候有一个细节，客户端在获取锁失败的情况下，会监听`/lock`节点。这会存在性能问题，如果在`/lock`节点下排队等待的有1000个进程，那么锁持有者释放锁(删除`/lock`节点下的临时节点)时，zookeeper会通知监听`/lock`的1000个进程。然后这1000个进程会读取zookeeper下`/lock`节点的全部临时节点，然后判断自己是否为最小的临时节点。但是这1000个进程中只有一个最小序号的进程会持有分布式锁，也就是说999个进程做的都是无用功。这些无用功会对zookeeper造成较大压力的读负载。为了解决惊群效应，需要对zookeeper分布式锁监听逻辑进行优化，实际上，排队进程真正感兴趣的是比自己临时节点序号小的节点，我们只需要监听序号比自己小的节点。

另外还需要注意的是，使用zookeeper方案也不是说就高枕无忧了。假设某一个客户端通过上述方案获得了锁，但由于网络问题，该客户端与zookeeper集群间失去了联系，zookeeper的心跳无效后，该客户端会收到了一个zookeeper的SessionTimeout的事件。为了保证分布式锁的有效性，这个时候客户端就需要在下一个等侯者获得锁之前，中断对共享资源的访问，然后继续尝试获取锁。

如果是javascript的项目，可以使用[zk-lock](https://github.com/metamx/zk-lock)库，它封装了上述方案的复杂逻辑，用起来也很方便。

## 四、三种方案方案比较

上面几种方式，哪种方式都无法做到完美。就像CAP一样，在复杂性、可靠性、性能等方面无法同时满足，所以，根据不同的应用场景选择最适合自己的才是王道。

- 从理解的难易程度角度（从低到高）
  - 数据库 > 缓存 > Zookeeper
- 从实现的复杂性角度（从低到高）
  - Zookeeper >= 缓存 > 数据库
- 从性能角度（从高到低）
  - 缓存 > Zookeeper >= 数据库
- 从可靠性角度（从高到低）
  - Zookeeper > 缓存 > 数据库

## 五、分布式锁实现

### 5.1 基于Redisson

#### 5.1.1 Redisson介绍

 大部分网站使用的分布式锁是基于缓存的，有更好的性能，而缓存一般是以集群方式部署，保证了高可用性。而Redis分布式锁官方推荐使用redisson。

Redisson原理图如下：

![Redisson原理图](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210731225633.png)

#### 5.1.2 Redisson锁说明

1、redission获取锁释放锁的使用和JDK里面的lock很相似，底层的实现采用了类似lock的处理方式
2、redisson 依赖redis，因此使用redisson 锁需要服务端安装redis，而且redisson 支持单机和集群两种模式下的锁的实现
3、redisson 在多线程或者说是分布式环境下实现机制，其实是通过设置key的方式进行实现，也就是说多个线程为了抢占同一个锁，其实就是争抢设置key。

#### 5.1.3 Redisson原理

**1)加锁：**

```lua
if (redis.call('exists', KEYS[1]) == 0) then 
        redis.call('hset', KEYS[1], ARGV[2], 1);
         redis.call('pexpire', KEYS[1], ARGV[1]); 
         return nil;
          end;
if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then
        redis.call('hincrby', KEYS[1], ARGV[2], 1);
        redis.call('pexpire', KEYS[1], ARGV[1]); 
        return nil;
        end;
return redis.call('pttl', KEYS[1]);
```

将业务封装在lua中发给redis，保障业务执行的原子性。

第1个if表示执行加锁，会先判断要加锁的key是否存在，不存在就加锁。

当第1个if执行，key存在的时候，会执行第2个if，第2个if会获取第1个if对应的key剩余的有效时间，然后会进入while循环，不停的尝试加锁。

**2)释放锁：**

```lua

if (redis.call('exists', KEYS[1]) == 0) then
       redis.call('publish', KEYS[2], ARGV[1]);
        return 1; 
        end;
if (redis.call('hexists', KEYS[1], ARGV[3]) == 0) then 
     return nil;
     end;
local counter = redis.call('hincrby', KEYS[1], ARGV[3], -1); 
if (counter > 0) then
     redis.call('pexpire', KEYS[1], ARGV[2]); 
     return 0; 
else redis.call('del', KEYS[1]); 
     redis.call('publish', KEYS[2], ARGV[1]); 
     return 1;
     end;
return nil;
```

执行lock.unlock(),每次都对myLock数据结构中的那个加锁次数减1。如果发现加锁次数是0了，说明这个客户端已经不再持有锁了，此时就会用：“del myLock”命令，从redis里删除这个key,另外的客户端2就可以尝试完成加锁了。

**3)缺点：**

Redis 的分布式锁在单实例的情况下是可以完美运行的，但是一旦涉及到 `reids`集群，就会出现重复加锁的情况。

Redisson存在一个问题，就是如果你对某个redis master实例，写入了myLock这种锁key的value，此时会异步复制给对应的master slave实例。但是这个过程中一旦发生redis master宕机，主备切换，redis slave变为了redis master。接着就会导致，客户端2来尝试加锁的时候，在新的redis master上完成了加锁，而客户端1也以为自己成功加了锁。此时就会导致多个客户端对一个分布式锁完成了加锁。这时系统在业务上一定会出现问题，导致脏数据的产生。

#### 5.1.4  Redisson配置与实现

##### **5.1.4.1 引入依赖**

```xml
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson-spring-boot-starter</artifactId>
    <version>3.11.0</version>
</dependency>
```

##### **5.1.4.2锁操作方法实现**

要想用到分布式锁，我们就必须要实现获取锁和释放锁，获取锁和释放锁可以编写一个`DistributedLocker`接口，代码如下：

```java
public interface DistributedLocker {
    /***
     * lock(), 拿不到lock就不罢休，不然线程就一直block
     * @param lockKey
     * @return
     */
    RLock lock(String lockKey);
    /***
     * timeout为加锁时间，单位为秒
     * @param lockKey
     * @param timeout
     * @return
     */
    RLock lock(String lockKey, long timeout);
    /***
     * timeout为加锁时间，时间单位由unit确定
     * @param lockKey
     * @param unit
     * @param timeout
     * @return
     */
    RLock lock(String lockKey, TimeUnit unit, long timeout);
    /***
     * tryLock()，马上返回，拿到lock就返回true，不然返回false。
     * 带时间限制的tryLock()，拿不到lock，就等一段时间，超时返回false.
     * @param lockKey
     * @param unit
     * @param waitTime
     * @param leaseTime
     * @return
     */
    boolean tryLock(String lockKey, TimeUnit unit, long waitTime, long leaseTime);
    /***
     * 解锁
     * @param lockKey
     */
    void unlock(String lockKey);
    /***
     * 解锁
     * @param lock
     */
    void unlock(RLock lock);
}
```

实现上面接口中对应的锁管理方法,编写一个锁管理类`RedissonDistributedLocker`，代码如下：

```java
@Component
public class RedissonDistributedLocker implements DistributedLocker {

    @Autowired
    private RedissonClient redissonClient;

    /***
     * lock(), 拿不到lock就不罢休，不然线程就一直block
     * @param lockKey
     * @return
     */
    @Override
    public RLock lock(String lockKey) {
        RLock lock = redissonClient.getLock(lockKey);
        lock.lock();
        return lock;
    }

    /***
     * timeout为加锁时间，单位为秒
     * @param lockKey
     * @param timeout
     * @return
     */
    @Override
    public RLock lock(String lockKey, long timeout) {
        RLock lock = redissonClient.getLock(lockKey);
        lock.lock(timeout, TimeUnit.SECONDS);
        return lock;
    }

    /***
     * timeout为加锁时间，时间单位由unit确定
     * @param lockKey
     * @param unit
     * @param timeout
     * @return
     */
    @Override
    public RLock lock(String lockKey, TimeUnit unit, long timeout) {
        RLock lock = redissonClient.getLock(lockKey);
        lock.lock(timeout, unit);
        return lock;
    }

    /***
     * tryLock()，马上返回，拿到lock就返回true，不然返回false。
     * 带时间限制的tryLock()，拿不到lock，就等一段时间，超时返回false.
     * @param lockKey
     * @param unit
     * @param waitTime
     * @param leaseTime
     * @return
     */
    @Override
    public boolean tryLock(String lockKey, TimeUnit unit, long waitTime, long leaseTime) {
        RLock lock = redissonClient.getLock(lockKey);
        try {
            return lock.tryLock(waitTime, leaseTime, unit);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return false;
    }

    /***
     * 解锁
     * @param lockKey
     */
    @Override
    public void unlock(String lockKey) {
        RLock lock = redissonClient.getLock(lockKey);
        lock.unlock();
    }

    /***
     * 解锁
     * @param lock
     */
    @Override
    public void unlock(RLock lock) {
        lock.unlock();
    }
}
```

##### **5.1.4.3配置Redis链接**

在resources下新建文件`redisson.yml`，主要用于配置redis集群节点链接配置，代码如下：

```yml
clusterServersConfig:
  # 连接空闲超时，单位：毫秒 默认10000
  idleConnectionTimeout: 10000
  pingTimeout: 1000
  # 同任何节点建立连接时的等待超时。时间单位是毫秒 默认10000
  connectTimeout: 10000
  # 等待节点回复命令的时间。该时间从命令发送成功时开始计时。默认3000
  timeout: 3000
  # 命令失败重试次数
  retryAttempts: 3
  # 命令重试发送时间间隔，单位：毫秒
  retryInterval: 1500
  # 重新连接时间间隔，单位：毫秒
  reconnectionTimeout: 3000
  # 执行失败最大次数
  failedAttempts: 3
  # 密码
  #password: test1234
  # 单个连接最大订阅数量
  subscriptionsPerConnection: 5
  clientName: null
  # loadBalancer 负载均衡算法类的选择
  loadBalancer: !<org.redisson.connection.balancer.RoundRobinLoadBalancer> {}
  #从节点发布和订阅连接的最小空闲连接数
  slaveSubscriptionConnectionMinimumIdleSize: 1
  #从节点发布和订阅连接池大小 默认值50
  slaveSubscriptionConnectionPoolSize: 50
  # 从节点最小空闲连接数 默认值32
  slaveConnectionMinimumIdleSize: 32
  # 从节点连接池大小 默认64
  slaveConnectionPoolSize: 64
  # 主节点最小空闲连接数 默认32
  masterConnectionMinimumIdleSize: 32
  # 主节点连接池大小 默认64
  masterConnectionPoolSize: 64
  # 订阅操作的负载均衡模式
  subscriptionMode: SLAVE
  # 只在从服务器读取
  readMode: SLAVE
  # 集群地址
  nodeAddresses:
    - "redis://192.168.211.137:7001"
    - "redis://192.168.211.137:7002"
    - "redis://192.168.211.137:7003"
    - "redis://192.168.211.137:7004"
    - "redis://192.168.211.137:7005"
    - "redis://192.168.211.137:7006"
  # 对Redis集群节点状态扫描的时间间隔。单位是毫秒。默认1000
  scanInterval: 1000
  #这个线程池数量被所有RTopic对象监听器，RRemoteService调用者和RExecutorService任务共同共享。默认2
threads: 0
#这个线程池数量是在一个Redisson实例内，被其创建的所有分布式数据类型和服务，以及底层客户端所一同共享的线程池里保存的线程数量。默认2
nettyThreads: 0
# 编码方式 默认org.redisson.codec.JsonJacksonCodec
codec: !<org.redisson.codec.JsonJacksonCodec> {}
#传输模式
transportMode: NIO
# 分布式锁自动过期时间，防止死锁，默认30000
lockWatchdogTimeout: 30000
# 通过该参数来修改是否按订阅发布消息的接收顺序出来消息，如果选否将对消息实行并行处理，该参数只适用于订阅发布消息的情况, 默认true
keepPubSubOrder: true
# 用来指定高性能引擎的行为。由于该变量值的选用与使用场景息息相关（NORMAL除外）我们建议对每个参数值都进行尝试。
#
#该参数仅限于Redisson PRO版本。
#performanceMode: HIGHER_THROUGHPUT
```

##### **5.1.4.4创建Redisson管理对象**

 Redisson管理对象有2个，分别为`RedissonClient`和`RedissonConnectionFactory`，我们只用在项目的`RedisConfig`中配置一下这2个对象即可，在`RedisConfig`中添加的代码如下：

```java
/****
 * Redisson客户端
 * @return
 * @throws IOException
 */
@Bean
public RedissonClient redisson() throws IOException {
    ClassPathResource resource = new ClassPathResource("redssion.yml");
    Config config = Config.fromYAML(resource.getInputStream());
    RedissonClient redisson = Redisson.create(config);
    return redisson;
}

/***
 * Redisson工厂对象
 * @param redisson
 * @return
 */
@Bean
public RedissonConnectionFactory redissonConnectionFactory(RedissonClient redisson) {
    return new RedissonConnectionFactory(redisson);
}
```

##### **5.1.4.5 测试代码**

```java

@RestController
@RequestMapping(value = "/redisson")
public class  RedissonController {

    @Autowired
    private RedissonDistributedLocker redissonDistributedLocker;

    /***
     * 多个用户实现加锁操作，只允许有一个用户可以获取到对应锁
     */
    @GetMapping(value = "/lock/{time}")
    public String lock(@PathVariable(value = "time")Long time) throws InterruptedException {
        System.out.println("当前休眠标识时间："+time);

        //获取锁
        RLock rlock = redissonDistributedLocker.lock("UUUUU");
        System.out.println("执行休眠："+time);

        Thread.sleep(time);

        System.out.println("休眠完成，准备释放锁："+time);
        //释放锁
        redissonDistributedLocker.unLocke(rlock);
        return "OK";
    }
}
```


## 六、结语

总的来说，目前分布式锁领域暂时没有出现十分完美、无懈可击的方案，Redis 以其高性能著称，但使用其实现分布式锁来解决并发仍存在一些困难。Redis 分布式锁只能作为一种缓解并发的手段，如果要完全解决并发问题，仍需要数据库的防并发手段。

## 七、扩展

### 7.1 分段锁

**ConcurrentHashMap**的源码和底层原理里面的核心思路，就是**分段加锁**！把数据分成很多个段，每个段是一个单独的锁，所以多个线程过来并发修改数据的时候，可以并发的修改不同段的数据。不至于说，同一时间只能有一个线程独占修改ConcurrentHashMap中的数据。

另外，Java 8中新增了一个LongAdder类，也是针对Java 7以前的AtomicLong进行的优化，解决的是CAS类操作在高并发场景下，使用乐观锁思路，会导致大量线程长时间重复循环。

LongAdder中也是采用了类似的分段CAS操作，失败则自动迁移到下一个分段进行CAS的思路。

![img](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210731231828)

假如现在iphone有1000个库存，可以给拆成20个库存段，例如可以在数据库的表里建20个库存字段，比如stock_01，stock_02，也可以在redis之类的地方放20个库存key。

接着，每秒1000个请求！此时可以写一个简单的随机算法，每个请求都是随机在20个分段库存里，选择一个进行加锁。

同时可以有最多20个下单请求一起执行，每个下单请求锁了一个库存分段，然后在业务逻辑里面，就对数据库或者是Redis中的那个分段库存进行操作即可，包括查库存 -> 判断库存是否充足 -> 扣减库存。

相当于一个20毫秒，可以并发处理掉20个下单请求，那么1秒，也就可以依次处理掉20 * 50 = 1000个对iphone的下单请求了。

一旦对某个数据做了分段处理之后，**要注意：就是如果某个下单请求，加锁后发现这个分段库存里的库存不足了，需要自动释放锁，然后立马换下一个分段库存，再次尝试加锁后尝试处理。这个过程一定要实现。**

不足肯定是有的，最大的不足，很不方便啊！实现太复杂了。

- 首先，需要对一个数据分段存储，一个库存字段本来好好的，现在要分为20个分段库存字段；
- 其次，需要每次处理库存的时候，还得自己写随机算法，随机挑选一个分段来处理；
- 最后，如果某个分段中的数据不足了，需要自动切换到下一个分段数据去处理。

这个过程都是要手动写代码实现的，还是有点工作量，挺麻烦的。

不过我们确实在一些业务场景里，因为用到了分布式锁，然后又必须要进行锁并发的优化，又进一步用到了分段加锁的技术方案，效果当然是很好的了，一下子并发性能可以增长几十倍。

## 八、参考

[石杉的架构笔记](https://juejin.cn/post/6844903719318847495)

