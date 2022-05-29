---
title: Git日常命令总结
top: false
cover: false
toc: true
mathjax: true
categories:
  - Git
  - 操作
tags:
  - Git
  - 操作
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
#创建dev和跟踪origin/dev --track 不加为缺省默认值
git checkout -b dev origin/dev
git checkout --track origin/dev
# 切换分支
git switch dev
git switch -c dev 0810beaed7
git switch -c <branch> --track <remote>/<branch>
```

# 3 Git Rebase 

```bash
# 将当前的commit 嫁接到新的base-commit上
git rebase <new base-commit> 
# 会先checkout到feature分支然后执行rebase master的操作
git rebase master feature
注意：rebase是会重写历史的，会导致和远端的分支分离
```

# 4 Git Rebase --onto

```bash
# A : 是一个分支名称（代表此分支的 HEAD）或者是一个 commit_id (此 id 不在 C 上)      
# B : 一个分支名称（此分支与 C 有共同的祖先 commit）或者是一个 commit_id (此 id 在 C 上)     
# C : 一个分支名称
git rebase --onto A B C 
```

**命令的作用：**

1. 首先会执行 git checkout 切换到 C

2.  将 B 到 C(HEAD) 之间所标识范围内的提交写到一个临时文件中 ，若B 为分支名称，则找到 B 与 C 共同的祖先 commit，则为此 commit 到 C(HEAD) 之间所标识范围内的提交，范围是（commit,C(HEAD)]。

3. 将当前分支强制重置（git reset --hard）到 A

4. 从2中临时文件的提交列表中，一个一个将提交按照顺序重新提交到重置之后的分支上

**注意：**

* 如果遇到提交已经在分支中包含，跳过该提交。
* 如果在提交过程遇到冲突，衍合过程暂停。用户解决冲突后，执行 git rebase --continue 继续变基操作。或者执行 git rebase --skip 跳过此提交。或者执行 git rebase --abort 就此终止变基操作切换到变基前的分支上。
* 衍合操作结束后，当前分支为 C

## 例

![原图片](https://raw.githubusercontent.com/lijinzedev/picture/main/img/20210702134539.png)

![执行后](https://raw.githubusercontent.com/lijinzedev/picture/main/img/20210702134615.png)

```bash
git rebase --onto master server client
```

**解释:**

​       server 和 client 的共同祖先commit 为 C3，首先当前分支切换到 client ，并将其 HEAD 重置为 master，即为 C6，然后将client 分支上 C3-C9 （不包括 C3，即 C8 和 C9）提交到当前分支上，即 C6 之后。最终结果即为 client : C1-C2-C5-C6-C8-C9，C8 和 C9 是重新提交，其 commit_id 会改变。

​        $ git checkout master 

​        $ git merge client 

​        再进行如上两步操作，即可将 client 分支的部分功能开发内容合入 master

# 5 Git Branch

```bash
#使用git branch -f 来移动分支指针
git branch -f master C2

git branch -u upstream/branch branch
```



## 扩展

> 在代码库中，我们如果希望从某一 ref 开始到 HEAD保留下来，然后之前的历史删除。因为这个任务比较常见，所以可以写成一个 shell script 

```bash
#!/bin/bash
git checkout --orphan temp $1
git commit -m "new begin"
git rebase --onto temp $1 master
git branch -D temp
```



# 5 参考

[Git Rebase](lixingcong.github.io/2019/12/04/git-rebase/)

[git rebase --onto ](https://www.zhihu.com/question/60279937)

[Git学习笔记（十） 改变历史 - Present - 博客频道 - CSDN.NET](https://link.zhihu.com/?target=http%3A//blog.csdn.net/agul_/article/details/7843182)
[如何批量删除git仓库中的提交纪录？ - SegmentFault](https://link.zhihu.com/?target=https%3A//segmentfault.com/q/1010000002564327)

[**git checkout --track原点/分支和git checkout -b分支原点/分支之间的区别**](http://blog.huati365.com/3qmg2N68XeAQNAK)