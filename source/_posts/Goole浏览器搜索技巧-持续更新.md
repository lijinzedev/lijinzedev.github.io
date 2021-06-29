---
title: '搜索技巧[持续更新]'
top: false
cover: false
toc: true
mathjax: true
categories:
  - 小技巧
tags:
  - 小技巧
date: 2020-12-06 21:03:39
password:
summary:
---

# GooGle

* 排除指定网站 `-site:blog.csdn.net`
* 指定网站 `site:blog.csdn.net`

# GitHub

## 指定搜索方式

* \<content\> in:file 搜索文件中有content的代码

* \<content\> in:path 搜索路径中有content的代码

* \<content\> language:java  搜索用java写的包含content的代码

* \<content\> in:fille,path  搜索路径或者是文件中有xxx的代码

  

## 通过语言搜索代码

* language:java     搜索用java写的代码

## 通过fork的数量或者是否有父节点的方式搜索

* boot language:java fork:true  搜索用Java写的boot关的代码并且被fork过

## 按照目录结构搜索

* controller path:src/main language:java 在*src/main* 目录下搜索controller关键字

## 通过文件名搜索

* filename:.vimrc commands 搜索 文件名匹配*.vimrc* 并且包含commands的代码

* minitest filename:test_helper path:test language:ruby 在test目录中搜索包含minitest且文件名匹配"*test_helper*"的代码

## 根据扩展名来搜索代码

*  form path:cgi-bin extension:pm    搜索cgi-bin目录下以pm为扩展名的代码





### 参阅

[Github官网](https://docs.github.com/en/github/searching-for-information-on-github/searching-on-github/searching-for-repositories)