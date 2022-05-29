---
title: Git Diff与Git Difftool
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
date: 2021-01-20 18:09:36
password:
summary:
---

# 配置Difftool



## 1 配置

### 1 meld

[官网](http://meldmerge.org/)

**命令配置**

```bash
git config --global diff.tool meld
git config --global difftool.prompt false
git config --global difftool.meld.cmd '"C:\Program Files (x86)\Meld\Meld.exe" "$LOCAL" "$REMOTE"'
git config --global merge.tool meld
git config --global mergetool.meld.cmd '"C:\Program Files (x86)\Meld\Meld.exe" "$BASE" "$LOCAL" "$REMOTE" "$MERGED"'
git config --global mergetool.bc3.trustExitCode true
```

**配置**

```bash
[difftool "meld"]
	cmd = \"C:\\Program Files (x86)\\Meld\\Meld.exe\" \"$LOCAL\" \"$REMOTE\"
[mergetool "meld"]
	cmd = \"C:\\Program Files (x86)\\Meld\\Meld.exe\" \"$BASE\" \"$LOCAL\" \"$REMOTE\" \"$MERGED\"
```

![效果](image-20210122110200827.png)



### 2 bc3

```bash

# 设置difftool
git config --global diff.tool bc3 
git config --global difftool.prompt false
# 设置mergetool
# cmd见下面设置
git config --global merge.tool bc3 
git config --global mergetool.bc3.trustExitCode true
# 在.gitconfig中添加
[merge]
	tool = bc3
[mergetool "bc3"]
    cmd = \"D:/workTools/Beyond Compare 4/bcomp.exe\" \"$LOCAL\" \"$REMOTE\" \"$BASE\" \"$MERGED\
# 在.gitconfig添加
[diff]
	tool = bc3
[difftool "bc3"]
    cmd = \"D:/workTools/Beyond Compare 4/bcomp.exe\" \"$LOCAL\" \"$REMOTE\"
```

![界面](image-20210122111632128.png)

### 3 p4merge

[官网](https://www.perforce.com/downloads/visual-merge-tool)

```bash
git config --global diff.tool p4merge
git config --global difftool.p4merge.path "C:\Program Files\Perforce\p4merge.exe"
# 因为每次使用diff tool的时候, git会弹出确认框, 我们最好把这个确认框从全局范围内默认不启用:
git config --global difftool.prompt false
git config --global merge.tool p4merge
git config --global mergetool.p4merge.path "C:\Program Files\Perforce\p4merge.exe"
git config --global mergetool.prompt false
```



## 2 使用

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

