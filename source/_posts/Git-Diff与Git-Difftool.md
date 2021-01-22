---
title: Git Diff与Git Difftool
top: false
cover: false
toc: true
mathjax: true
categories:
  - Git
tags:
  - Git
date: 2021-01-20 18:09:36
password:
summary:
---

# 配置Difftool

```bash

# 设置difftool
git config --global diff.tool bc3 
git config --global difftool.prompt false
# 在.gitconfig添加
[diff]
	tool = bc3
[difftool "bc3"]
    cmd = \"D:/workTools/Beyond Compare 4/bcomp.exe\" \"$LOCAL\" \"$REMOTE\"
```

## 1 使用

> 与 diff 参数相同

### 工作区与暂存区

> 当提示Launch 'bc'[Y/n] 的时候
>
> 临时解决方法：git difftool -y
>
> 永久解决方法：`git config --global difftool.prompt false`

* `git difftool --name-only` 

> 比较工作区与暂存区的文件，并列出变更列表

* `git difftool --no-symlinks`

> 依次在图像界面显示变更的文件

* `git difftool -d`

> 依次在图像界面显示变更的文件夹

### 暂存区与版本库

* `git difftool HEAD`

> 比较暂存区与HEAD

### 版本库与版本库

* `git diff commit commit`

# 配置MergeTool

```bash
# 设置mergetool
# cmd见下面设置
git config --global merge.tool bc3 
git config --global mergetool.bc3.trustExitCode true
# 在.gitconfig中添加
[merge]
	tool = bc3
[mergetool "bc3"]
    cmd = \"D:/workTools/Beyond Compare 4/bcomp.exe\" \"$LOCAL\" \"$REMOTE\" \"$BASE\" \"$MERGED\"
```



# git diff 命令详解

```bash
# 工作目录 vs 暂存区
git diff <filename>
git diff <branch> <filename>
# 暂存区 vs Git仓库
git diff --cached <filename>
git diff --cached <commit> <filename>
# 工作目录 vs Git仓库
git diff <commit> <filename>
# Git仓库 vs Git仓库
git diff <commit> <commit>
```

