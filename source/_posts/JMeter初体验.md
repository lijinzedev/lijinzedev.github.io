---
title: JMeter初体验
top: false
cover: false
toc: true
mathjax: true
categories:
  - 测试 - JMeter
tags:
  - JMeter
date: 2020-12-13 14:15:15
password:
summary:
---

# JMeter初体验

## 1 安装

### 1  安装Java环境变量（略…）

### 2 安装Jmeter

* [打开下载地址](http://jmeter.apache.org/download_jmeter.cgi)（Windows版本下载zip（Binaries），Linux版本下载tgz（Binaries））
* 解压
* 配置环境变量
  * 新增JMETER_HOME变量，变量值为解压路径
  * 编辑CLASSPATH变量加上 或者再Path中添加`%JMETER_HOME%\lib\ext\ApacheJMeter_core.jar;%JMETER_HOME%\lib\jorphan.jar;%JMETER_HOME%\lib\logkit-2.0.jar;`
  * 打开bin目录打开Jmeter.bat启动

## 2 吞吐量测试

1 创建用户组

![创建用户组](image-20201213143631332.png)

2 设置用户组参数

![设置用户组参数](image-20201213143930228.png)

3 添加HTTP请求

![添加HTTP请求](image-20201213144040565.png)

4 HTTP参数设置

![HTTP参数设置](image-20201213144317990.png)

5 HTTP的响应断言

![image-20201213144405613](image-20201213144405613.png)

6 响应断言设置

![image-20201213163712854](image-20201213163712854.png)


