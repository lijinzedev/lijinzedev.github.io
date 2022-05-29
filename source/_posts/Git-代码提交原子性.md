---
title: Git 代码提交原子性
top: false
cover: false
toc: true
mathjax: true
categories:
  - Git
  - 操作
tags:
  - Git
date: 2022-03-30 09:59:22
password:
summary:
---

# 1 git add -p 命令

* [教程文章](https://johnkary.net/blog/git-add-p-the-most-powerful-git-feature-youre-not-using-yet/)

* [了解 git diff命令](https://michael728.github.io/2020/06/14/git-diff-patch/)
* https://stackoverflow.com/questions/18142870/git-error-fatal-corrupt-patch-at-line-36
* https://stackoverflow.com/questions/1085162/commit-only-part-of-a-file-in-git

## 背景

> 当我们修改了代码准备提交时，本地的改动可能包含了不能提交的调试语句，还可能需要拆分成多个细粒度的 `pactch`。
>
> 使用 `git add -p` 来交互式选择代码片段，辅助整理出所需的 `patch`。

## 官方文档

```
-p, --patch
           Interactively choose hunks of patch between the index and the work tree and add them to
           the index. This gives the user a chance to review the difference before adding modified
           contents to the index.

           This effectively runs add --interactive, but bypasses the initial command menu and
           directly jumps to the patch subcommand. See “Interactive mode” for details.
```

```
-p, --patch
交互地在索引和工作树之间选择补丁块并将它们添加到索引中。这让用户有机会在将修改后的内容添加到索引之前查看差异。

这可以有效地运行 add --interactive，但是会绕过初始命令菜单，而直接跳转到 patch 子命令。有关详细信息，请参见`‘交互模式’'。
```

相关参数

```
y - 暂存此区块
n - 不暂存此区块
q - 退出；不暂存包括此块在内的剩余的区块
a - 暂存此块与此文件后面所有的区块
d - 不暂存此块与此文件后面所有的 区块
g - 选择并跳转至一个区块
/ - 搜索与给定正则表达示匹配的区块
j - 暂不决定，转至下一个未决定的区块
J - 暂不决定，转至一个区块
k - 暂不决定，转至上一个未决定的区块
K - 暂不决定，转至上一个区块
s - 将当前的区块分割成多个较小的区块
e - 手动编辑当前的区块
? - 输出帮助
```

