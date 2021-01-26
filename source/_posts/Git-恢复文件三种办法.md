---
title: Git 恢复文件三种办法
top: false
cover: false
toc: true
mathjax: true
categories:
  - Git
tags:
  - Git
date: 2021-01-26 10:00:42
password:
summary:

---



#  命令

> 这样直接覆盖工作区的文件了

* `git restore --source` 某次提交的commitdi -W 要写的文件名

> git cat 可以读取任意某次提交的文件内容，通过重定向到一个新文件，这样不影响现在工作区的修。

* `git cat-file commitid:文件的相对路径 > 新的文件名`

- `git reset 与 git checkout`
  - `reset hard`同时重置仓库 index 工作区
  - `reset mix`同时重置仓库 index
  - `reset soft`重置仓库
  - `checkout --` 这种用index重置工作区

# **结论**

只重置工作区，而且不是用index，是用仓库的提交，只能restore

但是这种操作都可能导致未提交的工作区修改丢失，因为没提交，所以不可逆

git cat 可以读取任意某次提交的文件内容，通过重定向到一个新文件，这样不影响现在工作区的修改，就曲线救国