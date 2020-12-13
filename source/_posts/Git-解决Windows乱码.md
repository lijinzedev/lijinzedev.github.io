---
title: 解决Windows Git乱码
top: false
cover: false
toc: true
mathjax: true
categories:
  - Git - 基础
tags:
  - Git - 基础
date: 2020-12-13 14:23:14
password:
summary:
---

# Windows Git乱码

* 第一步打开Git Bash 右键

![image-20201213142640209](image-20201213142640209.png)

* 修改 text为UTF-8

![image-20201213142722688](image-20201213142722688.png)

* 修改WIndows环境变量

  ```
  LESSCHARSET
  utf-8
  ```

  

![image-20201213142758974](image-20201213142758974.png)

* 输入一下命令配置全局变量

```bash
git config --global core.quotepath false
git config --global gui.encoding utf-8
git config --global i18n.commitEncoding utf-8
git config --global i18n.logoutputEncoding utf-8

# 或者修改.gitConfig
[gui]  
    encoding = utf-8  
    # 代码库统一使用utf-8  
[i18n]  
    commitencoding = utf-8  
    # log编码  
[svn]  
    pathnameencoding = utf-8  
    # 支持中文路径  
[core]
	quotepath = false 
	# status引用路径不再是八进制（反过来说就是允许显示中文了）
```

