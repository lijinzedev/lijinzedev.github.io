---
title: git 遇到紧急加塞任务怎么办? git stash入门
top: false
cover: false
toc: true
mathjax: true
categories:
  - Git
tags:
  - Git
date: 2021-01-26 10:34:30
password:
summary:
---

@[TOC](git stash)

**今作此文,寻章摘句,权抛砖引玉,遗笑方家处,敬请见谅**

> 场景: 平常我们在开发新的版本或者在探索一些奇妙的东西(手动滑稽)的时候,由于我们手上的的代码还没有生成commit,又没到生成commit的时候或者自己嫌麻烦懒得去做本地rebase了这时候 `git stash`就派生了用场  
# 常用命令
* `git stash` :执行存储不添加备注

* `git stash save "save message" ` : 在执行的存储上添加备注,避免当stash中的内容过多时造成混淆

* `git stash list`  ：查看当前stash中存在哪些存储

* `git stash show` ：显示做了哪些改动，默认show第一个存储,如果要显示其他存贮，使用 `git stash show stash@{$num}`

* `git stash show -p` : 显示第一个存储的改动，如果想显示其他存储,可以使用`git stash show  stash@{$num}  -p `

* `git stash apply` :应用某个存储,但不会把存储从存储列表中删除，如果要使用其他存储使用 `git stash apply stash@{$num}` **==注:这一条命令只会恢复工作区的内容,如果想恢复工作区和暂存区的内容使用下一条命令==**

* `git stash apply --index` :与前一条效果一样但是会多恢复暂存区

* `git stash pop stash@{序号} `：恢复保存列表里面指定的保存记录，并把恢复的记录从保存列表中删除 

* `git stash pop --index`与前一条效果一样但是会多恢复暂存区

* `git stash drop stash@{$num} `：删除stash指定保存的记录 不加@{$num} 默认第一条

* `git stash clear `：删除stash中所有的记录  

* `git stash push `

  - `-p|–patch`
    交互式stash，每个修改逐个确认，之后不停按y/n来选择要stash的修改
  - `-k|–[no-]keep-index`
    已经git add的文件stash之后修改还保留

  -k和--no-keep-index指定保存进度后，是否重置暂存区
  --patch 会显示工作区和HEAD的差异,通过编辑差异文件，排除不需要保存的内容。和git add -p命令类似

  - `-q|–quiet`
  - `-u|–include-untracked`
    新创建的文件直接git stash是不会被stash的，加上这个就可以
  - `-a|–all`
    比上一个-u更强，连被git ignore的文件都可以stash
  - `-m|–message <message>`
    指定一些说明性文字
  - `[--] [<pathspec>…]`
    指定stash的文件，但是如果其他文件有被git add过，也会被同时stash。不过只有指定的文件的修改会从工作副本中clean掉。

  ```bash
  # 指定说明信息并stash所有java文件的修改
  git stash push -m "this is a partly stash test" **/*.java
  ```

  * `git stash branch`

  > branch  [branchname]   [stash]
  >
  > 以这个stash被创建的那个commit为起点，创建一个叫branchname的分支，然后再在这个分支执行git stash pop —index stash

  * `git stash create`

  > 创建一个stash，并返回他的commit对象，但并不在refs中存储这个对象

  * `git stash store`

  > 存储通过create创建的stash。(可以在refs的stash和log/refs下看到这个stash)

# 示例
* 首先对工作区与暂存区都做了修改
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200405201855209.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxOTM0NDc4,size_16,color_FFFFFF,t_70)  
* 现在来了紧急任务保存现场 `git stash save "msg"`
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200405202202508.png)
* 当我们解决完了问题之后恢复之前的现场 这里的`\` 是一个转义字符
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200405202322751.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxOTM0NDc4,size_16,color_FFFFFF,t_70)  
* 不难发现 草(一种植物) 我暂存区内容的 emm 幸亏我们这里用了 apply git stash 中的记录还存在我们可以这样 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200405202557905.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxOTM0NDc4,size_16,color_FFFFFF,t_70)  
# 解决GIt Stash 冲突

```bash
    git stash branch new_branch [<stash>]  # <stash> will be the last one if not provided.
```



# 总结 

1.`git stash` 命令还是很好用的有木有

