---
title: IDEA 2021.1使用运行target
top: false
cover: false
toc: true
mathjax: true
categories:
  - idea
tags:
  - idea
date: 2021-04-19 17:04:35
password:
summary:
---

# IDEA在WSL2中运行Java程序

**前置条件**

* wsl2安装java环境，再点击wsl会自动识别，如果未安装则需要自己去设置一下环境，或者去查询一下

```shell
# 方法一：此方法是无法定位到java的安装路径的，只能定位到执行路径。
which java
# 方法二：
echo $JAVA_HOME
使用 echo $JAVA_HOME 命令可以定位到java安装路径，但前提是匹配了环境变量 $JAVA_HOME ，否则还是定位不到。
# 方法三：ls -lrt
# -a ：显示所有文件即目录（ls内定将文件名或目录名称开头为“.”的视为隐藏档，不会列出）
# -l： 除文件名称外，亦将文件形态、权限、拥有者、文件大小等资讯详细列出。
# -r： 将文件以相反次序显示（原定依英文字母次序）。
# -t： 将文件依次建立时间之先后次序列出。
# -A： 同-a，但不列出“.” (当前目录)及“…”(副文件)。
# -F: 在列出的文件名称后加一符号;例如可执行档则加“*”，目录则加“/”
# -R： 若目录下有文件，则以下之文件亦皆依序里列出。

```

1. [搭建wsl环境](https://lijinzedev.github.io/2021/04/12/win10-%E5%AE%89%E8%A3%85-wsl2/)

2. 更新Idea到2021.1
3. 选择运行目标

![change](change.gif)

![查找环境](image-20210419172711967.png)