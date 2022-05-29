---
title: SpringCloud项目搭建
top: false
cover: false
toc: true
mathjax: true
categories:
  - SpringCloud
  - 项目搭建
tags:
  - SpringCloud 
date: 2021-01-15 22:18:11
password:
summary:
---

# 一、新建工程

1. 创建Maven工程
2. 选择Maven版本
3. 设置字符编码

![设置字符编码](image-20210115222822095.png)

4. 注解激活生效

![注解激活生效](image-20210115222928116.png)

5. 选择java版本

![选择java版本](image-20210115222951511.png)

# 二、父工程POM文件配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.stu</groupId>
    <artifactId>app</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <!-- 统一管理jar包版本 -->
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <junit.version>4.12</junit.version>
        <log4j.version>1.2.17</log4j.version>
        <lombok.version>1.16.18</lombok.version>
        <mysql.version>5.1.47</mysql.version>
        <druid.version>1.1.16</druid.version>
        <mybatis.spring.boot.version>1.3.0</mybatis.spring.boot.version>
    </properties>

    <!-- 子模块继承之后，提供作用：锁定版本+子modlue不用写groupId和version  -->
    <dependencyManagement>
        <dependencies>
            <!--spring boot 2.2.2-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.2.2.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--spring cloud Hoxton.SR1-->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Hoxton.SR1</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--spring cloud alibaba 2.1.0.RELEASE-->
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>2.1.0.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>${mysql.version}</version>
            </dependency>
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid</artifactId>
                <version>${druid.version}</version>
            </dependency>
            <dependency>
                <groupId>org.mybatis.spring.boot</groupId>
                <artifactId>mybatis-spring-boot-starter</artifactId>
                <version>${mybatis.spring.boot.version}</version>
            </dependency>
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>${junit.version}</version>
            </dependency>
            <dependency>
                <groupId>log4j</groupId>
                <artifactId>log4j</artifactId>
                <version>${log4j.version}</version>
            </dependency>
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>${lombok.version}</version>
                <optional>true</optional>
            </dependency>
        </dependencies>
    </dependencyManagement>

</project>
```

# 三、dependencyManagement

## 1 介绍

> Maven使用dependencyManagement元素来提供了一种管理依赖版本号的方式，通常会在一个组织或者项目的最顶层的父POM中看到dependencyManagement元素。

## 2 作用

> 使用pom.xml中dependencyManagement元素能让所有在子项目中引用一个依赖而不用显式的列出版本号。Maven会沿着父子层向上走，直到找到一个拥有dependencyManagement元素的项目，然后他就会使用这个dependencyManagement中定义的版本号

## 3 好处

> 好处是如果多个子项目都引用同一个依赖，则可避免在每个使用的子项目中都声明版本号，这样当想升级或者切换到另一个版本的时候，只需要在顶层父容器里更新，而不需要一个一个的修改子项目，另外其他子项目如果需要另外一个版本，只需要生命Version即可

## 4 注意

* dependencyManagement里面只是声明依赖，`并不实现引入`，因此子项目需要显示的声明需要用到的依赖
* 如果不在子项目中声明依赖，是不会从父项目中继承下来的，只有在子项目中写了该依赖，才会从父项目中继承该项，并且version和scope都读取父pom
* 如果子项目中指定了版本号，那么就会使用子项目中指定的jar版本

# 四、注意

> 父工程创建完成执行mvn:install将父工程发布到仓库方便子工程继承