---
title: 重新认识IoC
top: false
cover: false
toc: true
mathjax: true
categories:
  - Spring编程核心思想
tags:
  - Spring编程核心思想
date: 2022-03-30 23:51:46
password:
summary:
---

# 一、IoC发展简介

## 1 什么是Ioc

> ​			In software engineering, inversion of control (IoC) is a programming principle. IoC inverts the flow of control as compared to traditional control flow. In IoC,custom-written portions of a computer program receive the flow of control from a generic framework. A software architecture with this design inverts control as compared to traditional procedural programming: in traditional programming, the custom code that expresses the purpose of the program calls into reusable libraries to take care of generic tasks, but with inversion of control, it is the framework that calls into the custom, or task-specific, code.

## 2 IoC发展简史

* 1983年，Richard E. Sweet 在《The Mesa Programming Environment》中提出“Hollywood Principle”（好莱坞原则） 

*  1988年，Ralph E. Johnson & Brian Foote 在《Designing Reusable Classes》中提出“Inversion of control”（控制反转） 

* 1996年，Michael Mattsson 在《Object-Oriented Frameworks, A survey of methodological issues》中将“Inversion of control”命名为 “Hollywood principle” 

* 2004年，Martin Fowler 在《Inversion of Control Containers and the Dependency Injection pattern》中提出了自己对 IoC 以及 DI 的理解 

* 2005年，Martin Fowler 在 《InversionOfControl》对 IoC 做出进一步的说明

# 二、IoC主要实现策略

**维基百科（https://en.wikipedia.org/wiki/Inversion_of_control）**

Implementation techniques 小节的定义：

“*In object-oriented programming, there are several basic techniques to implement inversion of control.*

*These are:*

* *Using a service locator pattern*
* *Using dependency injection, for example*
  * *Constructor injection*
  * *Parameter injection*
  * *Setter injection*
  * *Interface injection*
* *Using a contextualized lookup*
* *Using template method design pattern* 模板方法设计模式
* *Using strategy design pattern*

**《Expert One-on-One™ J2EE™ Development without EJB™》提到的主要实现策略： **

**“IoC is a broad concept that can be implemented in different ways. There are two main types:**

* *Dependency Lookup: The container provides callbacks to components, and a lookup context. This is* the EJB and Apache Avalon approach. It leaves the onus on each component to use container APIs* to look up resources and collaborators. The Inversion of Control is limited to the container* invoking callback methods that application code can use to obtain resources.
* *Dependency Injection: Components do no look up; they provide plain Java methods enabling the container to resolve dependencies. The container is wholly responsible for wiring up components,* *passing resolved objects in to JavaBean properties or constructors. Use of JavaBean properties* is called Setter Injection; use of constructor arguments is called Constructor Injection.”

# 三、IoC容器的职责

**维基百科（https://en.wikipedia.org/wiki/Inversion_of_control）**

在 Overview 小节中提到：

**“Inversion of control serves the following design purposes:**

* *To decouple the execution of a task from implementation.*

* *To focus a module on the task it is designed for.*

* *To free modules from assumptions about how other systems do what they do and instead rely on* *contracts.*

* *To prevent side effects when replacing a module.* 

  *Inversion of control is sometimes facetiously referred to as the "Hollywood Principle: Don't call* *us, we'll call you".”*

---

* 通用职责

  * 依赖处理
    * 依赖查找
    * 依赖注入
  * 生命周期管理
    * 容器
    * 托管的资源（Java Beans或其他资源）
  * 配置
    * 容器
    * 外部化配置
    * 托管的资源（Java Beans或其他资源）

  

# 四、IoC容器的实现

**主要实现**

* Java SE

  * Java Beans

  * Java ServiceLoader SPI

  * JNDI（Java Naming and Directory Interface）

* Java EE

  * EJB（Enterprise Java Beans）
  * Servlet

* 开源

  * Apache Avalon（http://avalon.apache.org/closed.html）

  * PicoContainer（http://picocontainer.com/）

  * Google Guice（https://github.com/google/guice）

  * Spring Framework（

    https://spring.io/projects/spring-framework）

  

# 五、传统IoC容器实现

# 六、轻量级IoC容器

# 七、依赖查找 VS 依赖注入

# 八、构造器注入VS Setter注入

# 九、面试题精选
