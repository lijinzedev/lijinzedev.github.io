---
title: JRebel热部署
top: false
cover: false
toc: true
mathjax: true
categories:
  - idea
tags:
  - idea
date: 2022-04-19 13:41:57
password:
summary:
---

# 官方文档

https://www.jrebel.com/products/jrebel/quickstart/intellij#installation

##  手动热部署项目

> 默认情况下，JRebel 热部署插件在你修改完已经编译好的 Java 文件失去焦点的时候，自动会将修改后 Java 文件编译，并替换掉旧的 Class 文件；

　　使用 Jrebel 热部署插件启动Tomcat项目，一般修改一两个Java文件，可能热部署会很慢，在失去焦点的时候才会自动编译已经修改后的Java文件，并替换旧的class文件，此时IDEA中并没有太多热部署重新编译替换这一系列操作的提示信息，你根本不知道是否已经替换成功！

　　**重点理解：Recompile、Rebuild、Build功能区别：**

　　　　a）Recompile：对选定的目标（Java 类文件），进行强制性编译，不管目标是否是被修改过。

　　　　b）Rebuild：对选定的目标（Project），进行强制性编译，不管目标是否是被修改过。由于 Rebuild 的目标只有 Project，所以 Rebuild 每次花的时间会比较长。

　　　　c）Build：对选定的目标（Project），编译那些被修改的文件；

　　**所以一般情况下，在使用热部署插件 JRebel 启动项目时，修改某个Java文件，手动的对项目进行热部署操作 Build -> Build Project**
