---
title: Git日常命令总结
top: false
cover: false
toc: true
mathjax: true
categories:
  - Git - 基础
tags:
  - Git - 基础
date: 2021-01-06 15:02:35
password:
summary:
---

# 1 Git 清除孤儿分支

```bash
# 假如你的远程版本库名是 origin,则使用如下命令先查看哪些分支需要清理：
git remote prune origin --dry-run
# 清理
git remote prune origin
# 删除本地
git branch -D feature_name1 feature_name2 feature_name3
```

# 2 Git创建分支

```bash
# 创建并切换分支
git checkout -b dev origin/dev
```

