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
# 切换分支
git switch dev
git switch -c dev 0810beaed7
```

# 3 Git Rebase 

```bash
# 将当前的commit 嫁接到新的base-commit上
git rebase <new base-commit> 
# 会先checkout到feature分支然后执行rebase master的操作
git rebase master feature

# 将当前的base节点，嫁接到新的base节点上
# 个人理解为将current base-commit ---> 当前分支所指向的commit 节点不包含current base-commit(左开右闭)
# 嫁接到新的节点 new base-commit 上
#  换句话说git执行这条命令的时候会先找到<current base-commit>和当前分支的共同祖先，然后将共同祖先之后的部分rebase到<new base-commit>。
git rebase --onto <new base-commit> <current base-commit>
# git执行这条命令的时候会先找到<current base-commit>和<target commit>的共同祖先，然后将共同祖先之后的部分rebase到<new base-commit>。
git rebase --onto <new base-commit> <current base-commit> <target commit>
```

## 示例

执行命令 `git rebase --onto  e6a161 a85b3fb`

![嫁接前](image-20210525151409346.png)

![嫁接后](image-20210525154731695.png)

# 参考文档

[Git Rebase](lixingcong.github.io/2019/12/04/git-rebase/)