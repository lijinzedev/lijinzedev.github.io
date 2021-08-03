---
title: Spring 事务传播机制使用
top: false
cover: false
toc: true
mathjax: true
categories:
  - Spring
tags:
  - Spring - 事务传播机制
date: 2021-08-02 22:07:40
password:
summary:
---

# Spring事务

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
|   PROPAGATION_MANDATORY   | `mandatory` 意为 `强制的` ，必须有事务，但是自己又不创建，如果调用者已经开启了事务，就用调用者的事务。调用者没开启事务，就抛异常。 | 不创建，如果没有事务就告警 |
| PROPAGATION_REQUIRES_NEW  | 如果已经有的话就把已经有的挂起。`两个事务是独立的,事务A中在提交事务B后如果 rollback 并不会导致事务B 的 rollback`。`需要使用 JtaTransactionManager作为事务管理器` | 挂起别人的，创建自己的     |
| PROPAGATION_NOT_SUPPORTED | not_supported` 即为 `不支持事务` , 调用者如果已有事务就把它挂起，自己以无事务运行。`需要使用 JtaTransactionManager作为事务管理器 | 挂起别人的，我不用         |
|     PROPAGATION_NEVER     |   `never` 从来都不需要事务的，如果调用者已有事务，就报错！   | 别人有就报错               |
|    PROPAGATION_NESTED     | nested` 嵌套的事务，自己会创建一个新事务，但是这个新事务并不是自己单独提交的，而是等待外层事务一起提交，所以事务B后面 事务A中的其他代码如果造成了 `rollback` 则也会导致事务B `rollback | 嵌套                       |

![事务](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210802225137.png)

## 二、环境搭建

### 2.1 数据库表

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

### 3.1  外围方法无事务

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

#### 3.1.1 结论

外围方法未开启事务，插入“joker”、“joker son”方法在自己的事务中独立运行，外围方法异常不影响内部插入“joker”、“joker son”方法独立的事务。

**在外围方法未开启事务的情况下`Propagation.REQUIRED`修饰的内部方法会新开启自己的事务，且开启的事务相互独立，互不干扰。**

### 3.2 外围方法有事务

**方法一：**

```java
/**     * 测试有父事务且抛出异常
     *
     * @param father
     * @param son
     */
@Transactional(propagation = Propagation.REQUIRED)
public void testWithTransaction(Father father, Son son) {
    fatherServiceImpl.add(father);
    sonServiceImpl.addAndThrowException(son);
}


```

>   “joker”、“joker son”均未插入。
>
>   外围方法开启事务，内部方法加入外围方法事务，外围方法回滚，内部方法也要回滚。

**方法二：**

```java
/**
     * 测试有父事务且抛出异常
     *
     * @param father
     * @param son
     */
@Transactional(propagation = Propagation.REQUIRED)
public void testWithTransactionAndThrow(Father father, Son son) {
    fatherServiceImpl.add(father);
    sonServiceImpl.add(son);
    throw new RuntimeException();
}
@Test
public void testWithTransactionAndThrow() throws Exception {
    final Father father = new Father();
    father.setId(1);
    father.setName("joker");
    final Son son = new Son();
    son.setId(1);
    son.setName("joker son ");
    testService.testWithTransactionAndThrow(father,son);
}
```

>   “joker”、“joker son”均未插入。
>
> 外围方法开启事务，内部方法加入外围方法事务，内部方法抛出异常回滚，外围方法感知异常致使整体事务回滚。

**方法三:**

```java
/**
     * 测试有父事务且抛出异常
     *
     * @param father
     * @param son
     */
@Transactional(propagation = Propagation.REQUIRED)
public void testWithTransactionAndTry(Father father, Son son) {
    fatherServiceImpl.add(father);
    try {
        sonServiceImpl.addAndThrowException(son);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
@Test
public void testWithTransactionAndTry() throws Exception {
    final Father father = new Father();
    father.setId(1);
    father.setName("joker");
    final Son son = new Son();
    son.setId(1);
    son.setName("joker son ");
    testService.testWithTransactionAndTry(father,son);
}
//抛出的异常是
//org.springframework.transaction.UnexpectedRollbackException: Transaction rolled back because it has been marked as rollback-only
```

>   “joker”、“joker son”均未插入。
>
> 外围方法开启事务，内部方法加入外围方法事务，内部方法抛出异常回滚，即使方法被catch不被外围方法感知，整个事务依然回滚。[详见官方文档](

分别执行验证方法，结果：

| 验证方法序号 | 数据库结果                     | 结果分析                                                     |
| ------------ | ------------------------------ | ------------------------------------------------------------ |
| 1            | “joker”、“joker son”均未插入。 | 外围方法开启事务，内部方法加入外围方法事务，外围方法回滚，内部方法也要回滚。 |
| 2            | “joker”、“joker son”均未插入。 | 外围方法开启事务，内部方法加入外围方法事务，内部方法抛出异常回滚，外围方法感知异常致使整体事务回滚。 |
| 3            | “joker”、“joker son”均未插入。 | 外围方法开启事务，内部方法加入外围方法事务，内部方法抛出异常回滚，即使方法被catch不被外围方法感知，整个事务依然回滚。[详见官方文档](https://docs.spring.io/spring-framework/docs/5.0.9.RELEASE/spring-framework-reference/data-access.html#tx-propagation-required) |

#### 3.2.1 结论

**结论：以上试验结果我们证明在外围方法开启事务的情况下`Propagation.REQUIRED`修饰的内部方法会加入到外围方法的事务中，所有`Propagation.REQUIRED`修饰的内部方法和外围方法均属于同一事务，只要一个方法回滚，整个事务均回滚。**

## 四、PROPAGATION_REQUIRES_NEW

**Service**

```java
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

    @Override
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void addRequiredNew(Father user) {
        baseMapper.insert(user);
    }

    @Override
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void addRequiredNewAndThrowException(Father user) {
        baseMapper.insert(user);
        throw new RuntimeException();
    }
}


@Service
@RequiredArgsConstructor
public class SonServiceImpl extends BaseServiceImpl<SonMapper, Son> implements Handler<Son> {


    @Override
    @Transactional(propagation = Propagation.REQUIRED)
    public void add(Son user) {
        baseMapper.insert(user);
    }

    @Override
    @Transactional(propagation = Propagation.REQUIRED)
    public void addAndThrowException(Son user) {
        baseMapper.insert(user);
        throw new RuntimeException();
    }

    @Override
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void addRequiredNew(Son user) {
        baseMapper.insert(user);
    }

    @Override
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void addRequiredNewAndThrowException(Son user) {
        baseMapper.insert(user);
        throw new RuntimeException();
    }
}

```

### 4.1  外围方法无事务

> 略

#### 4.1.1 结论

**在外围方法未开启事务的情况下`Propagation.REQUIRES_NEW`修饰的内部方法会新开启自己的事务，且开启的事务相互独立，互不干扰。**

### 4.2 外围方法有事务

**方法1： 外部方法抛出异常并且带有事物**

```java
/**
     * 外部方法抛出异常并且带有事物
     * @param father
     * @param son1
     * @param son2
     */
@Transactional(propagation = Propagation.REQUIRED)
public void transactionExceptionRequiredRequiresNewRequiresNew(Father father, Son son1, Son son2) {
    fatherServiceImpl.add(father);
    sonServiceImpl.add(son1);
    sonServiceImpl.addRequiredNew(son2);
    throw new RuntimeException();
}
@Test
public void transactionExceptionRequiredRequiresNewRequiresNew() throws Exception {
    final Father father = new Father();
    father.setId(1);
    father.setName("joker");
    final Son son1 = new Son();
    son1.setId(1);
    son1.setName("joker son ");
    final Son son2 = new Son();
    son2.setId(2);
    son2.setName("joker son and 2");
    testServiceRequiredNew.transactionExceptionRequiredRequiresNewRequiresNew(father, son1,son2);
}
```

> “joker”未插入，“joker son”未插入，“joker son and 2”插入。
>
> 外围方法开启事务，插入“joker”，“joker son”方法和外围方法一个事务，插入“joker son and 2”方法在独立的新建事务中，外围方法抛出异常只回滚和外围方法同一事务的方法，故插入“joker”，“joker son”的方法回滚。

**方法2： 外部方法有事物内部方法抛出异常**

```java
/**
     * 外部方法有事物内部方法抛出异常
     */
@Transactional(propagation = Propagation.REQUIRED)
public void transactionRequiredRequiresNewRequiresNewException(Father father, Son son1, Son son2) {
    fatherServiceImpl.add(father);
    sonServiceImpl.addRequiredNew(son1);
    sonServiceImpl.addRequiredNewAndThrowException(son2);
}

@Test
public void testTransactionRequiredRequiresNewRequiresNewException() throws Exception {
    final Father father = new Father();
    father.setId(1);
    father.setName("joker");
    final Son son = new Son();
    son.setId(1);
    son.setName("joker son ");
    final Son son2 = new Son();
    son2.setId(2);
    son2.setName("joker son and 2");
    testServiceRequiredNew.transactionRequiredRequiresNewRequiresNewException(father, son,son2);
}
```

>  “joker”未插入，“joker son ”插入，“joker son and 2”未插入。

> 外围方法开启事务，插入“joker”方法和外围方法一个事务，插入“joker son”方法、插入“joker son and 2”方法分别在独立的新建事务中。插入“joker son and 2”方法抛出异常，首先插入 “joker son and 2”方法的事务被回滚，异常继续抛出被外围方法感知，外围方法事务亦被回滚，故插入“joker”方法也被回滚。

**方法3： 外部方法有事物内部方法抛出异常且外部方法捉住异常**

```java
/**
     * 外部方法有事物内部方法抛出异常且外部方法捉住异常
     */
@Transactional(propagation = Propagation.REQUIRED)
public void  transactionRequiredRequiresNewRequiresNewExceptionTry(Father father, Son son1, Son son2) {
    fatherServiceImpl.add(father);
    sonServiceImpl.addRequiredNew(son1);
    try {
        sonServiceImpl.addRequiredNewAndThrowException(son2);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
@Test
public void testTransactionRequiredRequiresNewRequiresNewException() throws Exception {
    final Father father = new Father();
    father.setId(1);
    father.setName("joker");
    final Son son = new Son();
    son.setId(1);
    son.setName("joker son ");
    final Son son2 = new Son();
    son2.setId(2);
    son2.setName("joker son and 2");
    testServiceRequiredNew.transactionRequiredRequiresNewRequiresNewException(father, son,son2);
}
```

> “joker”插入，“joker son ”插入，“joker son and 2”未插入。

> 外围方法开启事务，插入“joker”方法和外围方法一个事务，插入“joker son”方法、插入“joker son and 2”方法分别在独立的新建事务中。插入“joker son and 2”方法抛出异常，首先插入“joker son and 2”方法的事务被回滚，异常被catch不会被外围方法感知，外围方法事务不回滚，故插入“joker”方法插入成功。

#### 4.2.1 结论

**结论：在外围方法开启事务的情况下`Propagation.REQUIRES_NEW`修饰的内部方法依然会单独开启独立事务，且与外部方法事务也独立，内部方法之间、内部方法和外部方法事务均相互独立，互不干扰。**

## 五、PROPAGATION_NESTED

#### 5.1  外围方法无事务

**方法一：**外围方法没有事物内部方法嵌套事物外部方法抛出异常

```java
    /**
     * 外围方法没有事物内部方法嵌套事物外部方法抛出异常
     *
     * @param father
     * @param son
     */

    public void withoutTransactionExceptionNestedNested(Father father, Son son) {
        fatherServiceImpl.addNested(father);
        sonServiceImpl.addNested(son);
        throw new RuntimeException();
    }

    @Test
    public void testWithoutTransactionExceptionNestedNested() throws Exception {
        final Father father = new Father();
        father.setId(1);
        father.setName("joker");
        final Son son = new Son();
        son.setId(1);
        son.setName("joker son ");
        testServiceNested.withoutTransactionExceptionNestedNested(father, son);
    }
```

> “joker”、“joker son”均插入。

> 外围方法未开启事务，插入“joker”、“joker  son”方法在自己的事务中独立运行，外围方法异常不影响内部插入“joker ”、“joker son”方法独立的事务。

**方法二：**外围方法没有事物内部方法嵌套事物内部方法抛出异常

```java
    /**
     * 外围方法没有事物内部方法嵌套事物内部方法抛出异常
     *
     * @param father
     * @param son
     */
    public void withoutTransactionNestedNestedException(Father father, Son son) {
        fatherServiceImpl.addNested(father);
        sonServiceImpl.addNestedException(son);
    }

    @Test
    public void testWithoutTransactionNestedNestedException() throws Exception {
        final Father father = new Father();
        father.setId(1);
        father.setName("joker");
        final Son son = new Son();
        son.setId(1);
        son.setName("joker son ");
        testServiceNested.withoutTransactionNestedNestedException(father, son);
    }
```

>  “joker”插入，“joker son”未插入。

>  外围方法没有事务，插入“joker”、“joker son”方法都在自己的事务中独立运行,所以插入“joker son”方法抛出异常只会回滚插入“joker son”方法，插入“joker”方法不受影响。

##### 5.1.1 总结

**结论：通过这两个方法我们证明了在外围方法未开启事务的情况下`Propagation.NESTED`和`Propagation.REQUIRED`作用相同，修饰的内部方法都会新开启自己的事务，且开启的事务相互独立，互不干扰。**

#### 5.2  外围方法有事务

**方法一:** 外部方法有事物 外部方法抛出异常

```java
/**
 * 外部方法有事物 外部方法抛出异常
 * @param father
 * @param son
 */
@Transactional(propagation = Propagation.REQUIRED)
public void transactionExceptionNestedNested(Father father, Son son) {
    fatherServiceImpl.addNested(father);
    sonServiceImpl.addNested(son);
    throw new RuntimeException();
}
@Test
public void testTransactionExceptionNestedNested() throws Exception {
    final Father father = new Father();
    father.setId(1);
    father.setName("joker");
    final Son son = new Son();
    son.setId(1);
    son.setName("joker son ");
    testServiceNested.transactionExceptionNestedNested(father, son);
}
```

> 均未插入。

> 外围方法开启事务，内部事务为外围事务的子事务，外围方法回滚，内部方法也要回滚。

**方法一:** 外部方法有事物 内部方法抛出异常

```java
    /**
     * 外部方法有事物 内部方法抛出异常
     * @param father
     * @param son
     */
    @Transactional(propagation = Propagation.REQUIRED)
    public void transactionNestedNestedException(Father father, Son son) {
        fatherServiceImpl.addNested(father);
        sonServiceImpl.addNestedException(son);
    }

    @Test
    public void testTransactionNestedNestedException() throws Exception {
        final Father father = new Father();
        father.setId(1);
        father.setName("joker");
        final Son son = new Son();
        son.setId(1);
        son.setName("joker son ");
        testServiceNested.transactionNestedNestedException(father, son);
    }
```

> 均未插入。

>  外围方法开启事务，内部事务为外围事务的子事务，内部方法抛出异常回滚，且外围方法感知异常致使整体事务回滚。

```java
/**
 * 外部方法有事物 内部方法抛出异常
 * @param father
 * @param son
 */
@Transactional(propagation = Propagation.REQUIRED)
public void transactionNestedNestedExceptionTry(Father father, Son son) {
    fatherServiceImpl.addNested(father);
    try {
        sonServiceImpl.addNestedException(son);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
    @Test
    public void testTransactionNestedNestedExceptionTry() throws Exception {
        final Father father = new Father();
        father.setId(1);
        father.setName("joker");
        final Son son = new Son();
        son.setId(1);
        son.setName("joker son ");
        testServiceNested.transactionNestedNestedExceptionTry(father, son);
    }
```

> “joker”插入、“joker son”未插入。

> 外围方法开启事务，内部事务为外围事务的子事务，插入“李四”内部方法抛出异常，可以单独对子事务回滚。

##### 5.2.1 总结

**结论：以上试验结果我们证明在外围方法开启事务的情况下`Propagation.NESTED`修饰的内部方法属于外部事务的子事务，外围主事务回滚，子事务一定回滚，而内部子事务可以单独回滚而不影响外围主事务和其他子事务**

## 六、REQUIRED  REQUIRES_NEW  与 NESTED区别

**NESTED和REQUIRED修饰的内部方法都属于外围方法事务，如果外围方法抛出异常，这两种方法的事务都会被回滚。但是REQUIRED是加入外围方法事务，所以和外围事务同属于一个事务，一旦REQUIRED事务抛出异常被回滚，外围方法事务也将被回滚。而NESTED是外围方法的子事务，有单独的保存点，所以NESTED方法抛出异常被回滚，不会影响到外围方法的事务。**

**NESTED和REQUIRES_NEW都可以做到内部方法事务回滚而不影响外围方法事务。但是因为NESTED是嵌套事务，所以外围方法回滚之后，作为外围方法事务的子事务也会被回滚。而REQUIRES_NEW是通过开启新的事务实现的，内部事务和外围事务是两个事务，外围事务回滚不会影响内部事务。**

## 七、总结

| Propagation   | Calling method (outer) | Called method (inner) |
| :------------ | :--------------------- | :-------------------- |
| REQUIRED      | No                     | T2                    |
| REQUIRED      | T1                     | T1                    |
| REQUIRES_NEW  | No                     | T2                    |
| REQUIRES_NEW  | T1                     | T2                    |
| MANDATORY     | No                     | Exception             |
| MANDATORY     | T1                     | T1                    |
| NOT_SUPPORTED | No                     | No                    |
| NOT_SUPPORTED | T1                     | No                    |
| SUPPORTS      | No                     | No                    |
| SUPPORTS      | T1                     | T1                    |
| NEVER         | No                     | No                    |
| NEVER         | T1                     | Exception             |
| NESTED        | No                     | T2                    |
| NESTED        | T1                     | T2                    |

## 参考

[Spring文档](https://docs.spring.io/spring-framework/docs/5.0.9.RELEASE/spring-framework-reference/data-access.html#tx-propagation-required)

[Spring-Framework 测试样例JDBC](https://github.com/spring-projects/spring-framework/blob/b595dc1dfad9db534ca7b9e8f46bb9926b88ab5a/spring-jdbc/src/test/java/org/springframework/jdbc/support/JdbcTransactionManagerTests.java)

[Spring-Framework 测试样例 DataSource](https://github.com/spring-projects/spring-framework/blob/b595dc1dfad9db534ca7b9e8f46bb9926b88ab5a/spring-jdbc/src/test/java/org/springframework/jdbc/datasource/DataSourceTransactionManagerTests.java)

[Don't Use @Transactional On Spring Boot Integration Test](http://aikin.me/2018/05/19/do-not-use-transactional-annotation-on-spring-integration-test/)

