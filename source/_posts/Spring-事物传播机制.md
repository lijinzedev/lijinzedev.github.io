---
title: Spring 事物传播机制使用
top: false
cover: false
toc: true
mathjax: true
categories:
  - Spring
tags:
  - Spring - 事物传播机制
date: 2021-08-02 22:07:40
password:
summary:
---

# Spring事物

## 一、前言

### 1.1 事务的传播

什么是事务的传播？传播，意味着是有两个事务参与的，单个事务是没有 ”传播“ 的概念的

事务的传播机制（`propagation behavior`），即为在一个事务方法中调用另一个事务方法时，事务该如何执行。

1.2 7种传播属性

​                                                                                                         

|     事务传播行为类型      |                             说明                             |                            |
| :-----------------------: | :----------------------------------------------------------: | -------------------------- |
|   PROPAGATION_REQUIRED    | `required` 含义是 `必须有事务`, 如果存在一个事务，则支持当前事务。如果没有事务则开启一个新的事务。 | 没有创建 `【默认】`        |
|   PROPAGATION_SUPPORTS    | `supports` 是支持事务，但是单独调用该方法时是没有事务的，如果调用者已经开启了事务，就用调用者的事务。 | 有就用，没有拉到           |
|   PROPAGATION_MANDATORY   | `mandatory` 意为 `强制的` ，必须有事务，但是自己又不创建，如果调用者已经开启了事务，就用调用者的事务。调用者没开启事务，就抛异常。 | 不创建，如果没有事物就告警 |
| PROPAGATION_REQUIRES_NEW  | 如果已经有的话就把已经有的挂起。`两个事务是独立的,事务A中在提交事务B后如果 rollback 并不会导致事务B 的 rollback`。`需要使用 JtaTransactionManager作为事务管理器` | 挂起别人的，创建自己的     |
| PROPAGATION_NOT_SUPPORTED | not_supported` 即为 `不支持事务` , 调用者如果已有事务就把它挂起，自己以无事务运行。`需要使用 JtaTransactionManager作为事务管理器 | 挂起别人的，我不用         |
|     PROPAGATION_NEVER     |   `never` 从来都不需要事务的，如果调用者已有事务，就报错！   | 别人有就报错               |
|    PROPAGATION_NESTED     | nested` 嵌套的事务，自己会创建一个新事务，但是这个新事务并不是自己单独提交的，而是等待外层事务一起提交，所以事务B后面 事务 A中的其他代码如果造成了 `rollback` 则也会导致事务B `rollback | 嵌套                       |

![事物](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210802225137.png)

## 二、环境搭建

### 1.1 数据库表

```mysql
CREATE TABLE `father` (
  `id` INTEGER UNSIGNED NOT NULL AUTO_INCREMENT,
  `name` VARCHAR(45) NOT NULL DEFAULT '',
  PRIMARY KEY(`id`)
) ENGINE = InnoDB;

CREATE TABLE `son` (
  `id` INTEGER UNSIGNED NOT NULL AUTO_INCREMENT,
  `name` VARCHAR(45) NOT NULL DEFAULT '',
  PRIMARY KEY(`id`)
) ENGINE = InnoDB;
```



## 三、PROPAGATION_REQUIRED

>  如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。这是最常见的选择。  

```java
@Service
@RequiredArgsConstructor
public class SonServiceImpl extends BaseServiceImpl<SonMapper, Son> implements Handler<Son> {


    @Override
    @Transactional(propagation = Propagation.REQUIRED)
    public void add(Son son) {
        baseMapper.insert(user);
    }

    @Override
    @Transactional(propagation = Propagation.REQUIRED)
    public void addAndThrowException(Son son) {
        baseMapper.insert(user);
        throw new RuntimeException();
    }
}

@Service
@RequiredArgsConstructor
public class FatherServiceImpl extends BaseServiceImpl<FatherMapper, Father> implements Handler<Father> {


    @Override
    @Transactional(propagation = Propagation.REQUIRED)
    public void add(Father father) {
        baseMapper.insert(father);
    }

    @Override
    @Transactional(propagation = Propagation.REQUIRED)
    public void addAndThrowException(Father father) {
  	    baseMapper.insert(father);
        throw new RuntimeException();
    }
}

```

****

### 1.1  外围方法无事物

```java
/**
 * 测试没有父事务且抛出异常
 * @param father
 * @param son
 */
public void testNotTransaction(Father father, Son son) {
    fatherServiceImpl.add(father);
    sonServiceImpl.addAndThrowException(son);
}
@Test
public void testNotTransaction() throws Exception {
    final Father father = new Father();
    father.setId(1);
    father.setName("joker");
    final Son son = new Son();
    son.setId(2);
    son.setName("joker son ");
    testService.testNotTransaction(father,son);
}
```

#### 1.1.1 结论

外围方法未开启事务，插入“joker”、“joker son”方法在自己的事务中独立运行，外围方法异常不影响内部插入“joker”、“joker son”方法独立的事务。

**在外围方法未开启事务的情况下`Propagation.REQUIRED`修饰的内部方法会新开启自己的事务，且开启的事务相互独立，互不干扰。**

### 1.2 外围方法有事物

**方法一：**

```java
/**     * 测试有父事务且抛出异常
     *
     * @param father
     * @param son
     */
@Transactional(propagation = Propagation.REQUIRED)
public void testHaveTransaction(Father father, Son son) {
    fatherServiceImpl.add(father);
    sonServiceImpl.addAndThrowException(son);
}


```

**方法二：**

```java
/**
     * 测试有父事务且抛出异常
     *
     * @param father
     * @param son
     */
@Transactional(propagation = Propagation.REQUIRED)
public void testHaveTransactionAndThrow(Father father, Son son) {
    fatherServiceImpl.add(father);
    sonServiceImpl.add(son);
    throw new RuntimeException();
}
@Test
public void testHaveTransactionAndThrow() throws Exception {
    final Father father = new Father();
    father.setId(1);
    father.setName("joker");
    final Son son = new Son();
    son.setId(1);
    son.setName("joker son ");
    testService.testHaveTransactionAndThrow(father,son);
}
```

**方法三:**

```java
/**
     * 测试有父事务且抛出异常
     *
     * @param father
     * @param son
     */
@Transactional(propagation = Propagation.REQUIRED)
public void testHaveTransactionAndTry(Father father, Son son) {
    fatherServiceImpl.add(father);
    try {
        sonServiceImpl.addAndThrowException(son);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
@Test
public void testHaveTransactionAndTry() throws Exception {
    final Father father = new Father();
    father.setId(1);
    father.setName("joker");
    final Son son = new Son();
    son.setId(1);
    son.setName("joker son ");
    testService.testHaveTransactionAndTry(father,son);
}
//抛出的异常是
//org.springframework.transaction.UnexpectedRollbackException: Transaction rolled back because it has been marked as rollback-only
```

分别执行验证方法，结果：

| 验证方法序号 | 数据库结果               | 结果分析                                                     |
| ------------ | ------------------------ | ------------------------------------------------------------ |
| 1            | “张三”、“李四”均未插入。 | 外围方法开启事务，内部方法加入外围方法事务，外围方法回滚，内部方法也要回滚。 |
| 2            | “张三”、“李四”均未插入。 | 外围方法开启事务，内部方法加入外围方法事务，内部方法抛出异常回滚，外围方法感知异常致使整体事务回滚。 |
| 3            | “张三”、“李四”均未插入。 | 外围方法开启事务，内部方法加入外围方法事务，内部方法抛出异常回滚，即使方法被catch不被外围方法感知，整个事务依然回滚。详见https://docs.spring.io/spring-framework/docs/5.0.9.RELEASE/spring-framework-reference/data-access.html#tx-propagation-required |

#### 1.2.1 结论

**结论：以上试验结果我们证明在外围方法开启事务的情况下`Propagation.REQUIRED`修饰的内部方法会加入到外围方法的事务中，所有`Propagation.REQUIRED`修饰的内部方法和外围方法均属于同一事务，只要一个方法回滚，整个事务均回滚。**

## 四、PROPAGATION_REQUIRES_NEW

## 参考

[Spring文档](https://docs.spring.io/spring-framework/docs/5.0.9.RELEASE/spring-framework-reference/data-access.html#tx-propagation-required)

https://www.jianshu.com/p/25c8e5a35ece

http://cxyzjd.com/article/PitBXu/114555573

https://blog.csdn.net/soonfly/article/details/70305683

https://juejin.cn/post/6844903600943022088

https://processon.com/diagraming/6107fa916376897465d52346

https://segmentfault.com/a/1190000022620219

https://blog.51cto.com/u_15050718/2623204

https://fgu123.github.io/2019/03/19/Spring-Transaction-Propagation/