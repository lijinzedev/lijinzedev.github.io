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

​                                                                                                         

|     事务传播行为类型      |                             说明                             |
| :-----------------------: | :----------------------------------------------------------: |
|   PROPAGATION_REQUIRED    | 如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。这是最常见的选择。 |
|   PROPAGATION_SUPPORTS    |     支持当前事务，如果当前没有事务，就以非事务方式执行。     |
|   PROPAGATION_MANDATORY   |        使用当前的事务，如果当前没有事务，就抛出异常。        |
| PROPAGATION_REQUIRES_NEW  |         新建事务，如果当前存在事务，把当前事务挂起。         |
| PROPAGATION_NOT_SUPPORTED |  以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。  |
|     PROPAGATION_NEVER     |       以非事务方式执行，如果当前存在事务，则抛出异常。       |
|    PROPAGATION_NESTED     | 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作。 |

# 一、环境搭建

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



## 二、PROPAGATION_REQUIRED

>  如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。这是最常见的选择。  

### 1.1  外围方法无事物





### 2.2 