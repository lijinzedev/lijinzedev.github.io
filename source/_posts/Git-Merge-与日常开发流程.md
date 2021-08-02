---
title: Git Merge
top: false
cover: false
toc: true
mathjax: true
categories:
  - Git
tags:
  - Git
date: 2021-07-29 10:05:38
password:
summary:
---

# 一、Git Merge 

![image-20210729143535128](https://raw.githubusercontent.com/lijinzedev/picture/main/img/20210729143535.png)

## 参数

* `--commit`和`--no-commit`

> `--commit`参数使得合并后产生一个合并结果的commit节点。该参数可以覆盖`--no-commit`。
> `--no-commit`参数使得合并后，为了防止合并失败并不自动提交，能够给使用者一个机会在提交前审视和修改合并结果。

* `--squash`

> 根据字面意思，这个操作完成的是压缩的提交；解决的是什么问题呢，由于在dev分支上执行的是开发工作，有一些很小的提交，或者是纠正前面的错误的提交，对于这类提交对整个工程来说不需要单独显示出来一次提交，不然导致项目的提交历史过于复杂；所以基于这种原因，我们可以把dev上的所有提交都合并成一个提交；然后提交到主干。

### 使用条件:

> 判断是否使用`--squash`选项最根本的标准是，待合并分支上的历史是否有意义。