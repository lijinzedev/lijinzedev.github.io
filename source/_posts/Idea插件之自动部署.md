---
title: Idea插件之自动部署
top: false
cover: false
toc: true
mathjax: true
categories:
  - idea
tags:
  - idea
date: 2021-01-16 21:01:35
password:
summary:
---



# 1 下载

> 下载alibaba cloud toolkit工具

# 2 配置

## 1 添加Host

![添加Host](image-20210116210439432.png)

## 2 配置账号密码

![配置账号密码](image-20210116210601635.png)

## 3 补充tag与描述

![补充tag与描述](image-20210116210706379.png)

## 4 添加命令

```bash
# 根据进程名找到进程Id停掉历史进程
ps -aux | grep -v grep | grep ${serverName} | awk '{print $2}' | xargs kill -9
# 启动新上传的java jar 应用服务
nohup java -jar /root/${serverName}.jar &
```

![添加命令](image-20210116213132180.png)

![添加命令](image-20210116213104902.png)

## 5 应用部署配置

需要部署的项目右键->Alibaba Cloud -> Deploy To Host

![应用部署配置](image-20210116213510808.png)

- 本次部署配置的名称：Name，配置固化下来之后可以复用
- 在项目上传到服务器之前[maven](http://www.zimug.com/tag/maven)打包：Maven Build。也可以选择使用Gradle打包：Gradle Build或者手动打包之后上传文件：Upload File。
- 选择远程部署的服务器的Ip，本文中第二步的配置结果
- Target Directory：[maven](http://www.zimug.com/tag/maven)打包之后的文件上传目录（即应用部署目录）：根据自己的主机路径规划填写。
- After Deploy：当文件上传主机之后执行的shell脚本或命令行，我们这里选择执行`nohup java -jar /root/server-jwt-1.0.jar &;`启动应用。
- Run Maven Goal :[maven](http://www.zimug.com/tag/maven) 的打包目标，先对父项目打包，再对子模块打包。如果不存在，就点击“+”新建，打包命令是“clean install”

![image-20210116213606080](image-20210116213606080.png)

除了应用打包、上传、启动之外，我们通常需要一些额外的动作。

- 比如：之前已将发过一版，再次部署发版应该先把旧版本进程停掉。选择`ps -aux|grep -v grep |grep server-jwt| awk '{print $2}'|xargs kill -9;`命令行，第三步配置好的。
- 比如：应用部署完成之后，应该立刻查看应用启动的日志，观察是否正常。

![添加日志](image-20210116213638558.png)

# 3 参阅

[项目部署](http://www.zimug.com/java/%e9%a1%b9%e7%9b%ae%e9%83%a8%e7%bd%b2%e7%82%b9%e4%b8%80%e4%b8%8b%e6%8c%89%e9%92%ae%e5%b0%b1%e5%8f%af%e4%bb%a5%ef%bc%8c%e5%85%a8%e6%b5%81%e7%a8%8b%e8%87%aa%e5%8a%a8%e5%8c%96-%e4%b8%89%e5%88%86%e9%92%9f/.html)