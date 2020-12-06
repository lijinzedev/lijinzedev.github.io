---
title: Spring 事物传播机制使用
top: false
cover: false
toc: true
mathjax: true
categories:
  - Spring - 事物
tags:
  - Spring - 事物传播机制
date: 2020-12-03 22:07:40
password:
summary:
---

# 1 Spring事物



# 1 使用用例

```mysql
CREATE TABLE `user1` (
  `id` INTEGER UNSIGNED NOT NULL AUTO_INCREMENT,
  `name` VARCHAR(45) NOT NULL DEFAULT '',
  PRIMARY KEY(`id`)
)
ENGINE = InnoDB;

CREATE TABLE `user2` (
  `id` INTEGER UNSIGNED NOT NULL AUTO_INCREMENT,
  `name` VARCHAR(45) NOT NULL DEFAULT '',
  PRIMARY KEY(`id`)
)
ENGINE = InnoDB;
```

