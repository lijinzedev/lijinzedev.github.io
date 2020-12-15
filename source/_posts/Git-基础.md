---
title: Git 基础
top: false
cover: false
toc: true
mathjax: true
categories:
  - Git - 基础
tags:
  - Git - 基础
date: 2020-11-30 11:07:32
password:
summary:

---



[toc]

# git命令补充学习

## git更新.gitignore

```bash
git rm -r --cached .//清空缓存
git add .//重新提交
git commit -m "update .gitignore"
git push


```



## git设置别名

### 全局

```bash
alias gs="git status"
alias ga="git add ."
alias gc="git commit -m"
alias gl="git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr)%Creset | %C(bold)%cn' --abbrev-commit --date=relative"
alias gck="git checkout"
alias gcb="git checkout -b"
alias gb="git branch"
alias gm="git merge"
alias grb="git rebase"
alias grs="git reset HEAD ."
alias gca="git commit --amend"
```

### 非全局

```bash
git config  --global alias.st status
```

## git tag以及log的命令技巧

# Git初学

## 0 常用命令解析

```bash
# 他会监控工作区的状态树，使用它会把工作时的所有变化提交到暂存区，包括文件内容修改(modified)以及新文件(new)，但不包括被删除的文件。
git add .

# 他仅监控已经被add的文件（即tracked file），他会将被修改的文件提交到暂存区。add -u 不会提交新文件（untracked file）。（git add --update的缩写）
git add -u

# 是上面两个功能的合集（git add --all的缩写）
git add -A

git reset

# 查看本地分支
git branch -v
# 查看本地及其远端所有分支
git branch -va
```

## 1 使用git需要做的小配置

```bash
git config --global user.name 'your_name'
git config --global user.email 'your_email'
```

### 1. Config 的三个作用域

```bash
# 只对某个仓库有效
git config --local
# 对当前用户的所有仓库有效
git config --global
# 对系统所有登录的账户有效
git config --system

# 显示 config 的配置， 家--list
git config --list --local
git config --list --global
git config --list --system
```

## 2 工作区与暂存区

![image-20200211143954709](http://ww1.sinaimg.cn/large/006vQ8sXgy1gfioj09omlj30ju08n0ud.jpg?ynotemdtimestamp=1606703976531)

![image-20200211144014011](http://ww1.sinaimg.cn/large/006vQ8sXgy1gfioj08peqj30la08ota9.jpg?ynotemdtimestamp=1606703976531)

![image-20200211144021875](http://ww1.sinaimg.cn/large/006vQ8sXgy1gfioj07z2ej30ig08vta6.jpg?ynotemdtimestamp=1606703976531)

## 3 给文件重命名

```bash
# 普通方法
mv readme readme.md
git add readme.md
git rm readme
# 简便的方法
git mv readme readme.md
# 查看命令详情
git log --help --web
# 查看类型
git cat-file -t 3d4731d
# 查看内容
git cat-file -p 3d4731d
```

## 4 gitk 图形界面

## 5 分离头指针

> 当在checkout 一个 commit 的时候 产生分离头指针
>
> 使用场景： 做尝试性的变更使用分离头指针
>
> 分离头指针指的是：变更没基于某个branch去做 做分支切换的时候 分离头指针产生的commit 可能会被git当做垃圾清理掉，切记要和branch 绑定在一起

## 6 查看版本历史

```bash
# 查看简单的历史
git log --oneline
# 查看最近几个简单的历史
git log --n2 --oneline
# 图形化查看
git log --all --graph
```

## 7 HEAD

> HEAD 指向commit 或者指向分离头指针的commit

```bash
# 比较两个commit之间的差异
git diff HEAD HEAD^
```

## 8 git原理探秘

> git 的对象 commit tree blob

- config目录：仓库配置文件
- HEAD
- objects：git中只要任何文件中文件内容相同那么他就是唯一的blob
- refs :引用/分支
  - heads：分支 内容为 commit
  - tags: 标签 内容为 commit
- commit tree blob 的关系

![image-20200211215858843](http://ww1.sinaimg.cn/large/006vQ8sXgy1gfiootxqaoj30ky0cp0yh.jpg?ynotemdtimestamp=1606703976531)

- 新建一个 仓库

  ```
  git init  test
  ```

  ![image-20200211221038828](http://ww1.sinaimg.cn/large/006vQ8sXgy1gfioq4rohdj30jn0a6gm8.jpg?ynotemdtimestamp=1606703976531)

  ```bash
  # 创建一个文件
  echo "Hello World" >> readme.md
  # 查看 objects 文件下是否有东西
  find .git/objects/ -type f  
  # 添加到暂存区
  git add .
  ```

  ![image-20200211222153328](http://ww1.sinaimg.cn/large/006vQ8sXgy1gfioqjlvpyj30k30bpdgv.jpg?ynotemdtimestamp=1606703976531)

  

## 9 删除不需要的分支

```bash
git branch -d 分支名
# 小d删除不了用D
git branch -D 分支名 
```

## 10 修改commit的message

```bash
# 修改最新commit的message
git commit --amend
# 修改老旧的commit的message
# 变基要选择 被变得父亲的commit
git rebase -i 
```

![image-20200212192200334](http://ww1.sinaimg.cn/large/006vQ8sXgy1gfiosta3ztj30kb0cht9t.jpg?ynotemdtimestamp=1606703976531)

> 将变更的 第一个 7c6e74b 前面的 pick 改为 reword（或者偷懒谢一个 r） 保存 退出 git 会自己弹出另外一个 交互的界面

![image-20200212192418247](http://ww1.sinaimg.cn/large/006vQ8sXgy1gfiq4m4w7cj30kb0chgmh.jpg?ynotemdtimestamp=1606703976531)

> 保存退出 出现successfull 就成功了
>
> 这里用到了分离头指针 然后做调整 然后 master与head 指向最新的 commit
>
> 变基的行为：已经贡献到集成的分支上 建议不要 否则会影响其他同事

## 11 如何将几个commit合并为一个

- 将连续的commit合并

```bash
# 如果合并前四个那么就选择 第五个的 commit
git rebase -i 第五个
# 基于哪个一个合道哪一个commit上去
```

> 那些commit 变更的文件都要但是要把他合并到前面的commit上去

![image-20200212210308978](http://ww1.sinaimg.cn/large/006vQ8sXgy1gfiq546m6cj30kp0ch0tw.jpg?ynotemdtimestamp=1606703976531)

![image-20200212224256596](http://ww1.sinaimg.cn/large/006vQ8sXgy1gfiq5ft0o1j30gp04l407.jpg?ynotemdtimestamp=1606703976531)

- 将不连续的 commit 合并

  合并两个 添加readme的 commit

  > 将最古老的 commit
  >
  > git rebase -i 最古老的

![image-20200212225139935](http://ww1.sinaimg.cn/large/006vQ8sXgy1gfiq5q6revj30pi0eqdl7.jpg?ynotemdtimestamp=1606703976531)

> 发现不够手动添加最古老的
>
> pick 最古老的commit
>
> 然后调整pick ’commit‘ 的位置 吧 需要合并的挨着放

![image-20200212225549125](http://ww1.sinaimg.cn/large/006vQ8sXgy1gfiq6g0g7lj316f0ksqb7.jpg?ynotemdtimestamp=1606703976531)

![image-20200212225725697](http://ww1.sinaimg.cn/large/006vQ8sXgy1gfiq6rbkphj30m0034761.jpg?ynotemdtimestamp=1606703976531)

> 输入 git statue 然后根据提示 接着做
>
> 变基完成

![image-20200212230015054](http://ww1.sinaimg.cn/large/006vQ8sXgy1gfiq77wxwmj30up0g6gsm.jpg?ynotemdtimestamp=1606703976531)

> 出现了两个 commit 没有祖先的
>
> 但是和master 无关的分支已经没啥用##了

![image-20200212230320205](http://ww1.sinaimg.cn/large/006vQ8sXgy1gfiq7ha4sgj30a107gmy5.jpg?ynotemdtimestamp=1606703976531)

## 12 如何比较暂存区和HEAD所含文件的差异

```bash
git diff --cached
```

![image-20200212230934332](http://ww1.sinaimg.cn/large/006vQ8sXgy1gfiq8dedjtj30kz09x0vd.jpg?ynotemdtimestamp=1606703976531)

## 13 如何比较工作区和暂存区的差异

```bash
# 工作区与暂存区所有的差别
git diff
# 工作区与暂存区 某些文件的差别 
git diff -- readme style/style.css
```

![image-20200212231543413](http://ww1.sinaimg.cn/large/006vQ8sXgy1gfiq8obmbvj30gc0bvgr0.jpg?ynotemdtimestamp=1606703976531)

## 14 如何让暂存区恢复和HEAD一样

> 场景：不想保留暂存区

```bash
# 将暂存区 恢复成和HEAD一样
git reset HEAD
# 比较暂存区与HEAD 没有输出 就表示一致
git diff --cached
```

## 15 如何将工作区回复和暂存区的状态

```bash
# 变更工作区使用checkout
git checkout -- '文件名'
```

## 16 如何取消暂存区部分文件的修改

```bash
# git reset HEAD -- '具体的文件可以多个'
git reset HEAD -- styles/style.css readme.md
```

## 17 消除最近的几次提交

```bash
# 暂存区和工作区都恢复到指定的内容了
git reset --hrad ‘commit’
```

## 18 看看不同提交的指定文件差异

```bash
# 分支即为指针 (显示所有)
git diff temp master
# 显示感兴趣的文件
git diff temp master -- '文件名'
# 也可以用 master 对应的 commit 和 temp 对应的commit
git branch -av
git diff 'commmit' 'commit'
```

## 19 正确删除文件的方法

```bash
# 普通方法
# 工作目录下删除
rm readme
# 暂存区删除
git rm readme 

# 简单方法 
# 直接把删除文件放到暂存区
git rm ‘file_name’
```

## 20 开发中临时加塞了紧急任务怎么处理

> 场景：正在做新功能的开发但是开发未结束，其他分支产生了bug 需要去解决

```bash
# 使用 stash 
git stash 
# 查看 stash 信息
# stash 相当于 堆栈
git stash list
# 去执行紧急任务
# 紧急任务修复完成
# 恢复 使用 apply 或者pop
# apply两个作用：
1 把之前的内容弹出来 放入工作区
2 stash 的信息还在可以反复使用
git stash apply
# pop  把之前的内容弹出来 放入工作区
# 与栈的pop一样
```

## 21 如何制定不需要git管理的文件

> - .gitignore 文件
>
>   **a.忽略指定文件/目录**
>
> - 忽略指定文件
>
>   - HelloWrold.class
>
> - 忽略指定文件夹
>
>   - bin/
>
> - bin/gen/
>
>   **b.通配符忽略规则**
>
> - 忽略.class的所有文件 *.class
>
> - 忽略名称中末尾为ignore的文件夹
>
> - *ignore/
>
> - 忽略名称中间包含ignore的文件夹
>
>   - *ignore*/

## 22 Git 的备份

![image-20200213122814992](http://ww1.sinaimg.cn/large/006vQ8sXgy1gfiq94we8ij30ql0i1td0.jpg?ynotemdtimestamp=1606703976531)

> **直观区别：** 哑协议传输进度不可见；智能传输协议可见
>
> **传输速度:** 智能协议比哑协议传输速度快

```bash
# 克隆不带工作区的 裸仓库
git clone --bare ‘协议地址’
# 哑协议
git clone --bare /d/00_aa_lijinze_java_web_stady/gitsu
# 智能协议 带进度显示 并且更快
git clone --bare file:///d/00_aa_lijinze_java_web_stady/gitsu
# 与远端仓库同步
# -v 查看remote
git remote -v
# 删除remote
git remote remove 'name'
# 创建 remote
git remote add zhineng  file:///d/00_aa_lijinze_java_web_stady/gitsu
# 格式
git remote add 'name' 'address'
# 更新到远端
git push 'remote_name' 'branch_name'
```

## 23 配置公钥

> 在github上面的help 搜索ssh即可

## 24 push到GitHub上面去

> 首先复制ssh链接

```bash
# 将本地所有的分支push 到 远程去
git push origin --all
# git pull
1. fetch 把远端分支的内容拉下来
2. 将本地的分支与远端的分支marge
# 将本地的 master 与远端的master合并
git marge origin/master
# 由于是不想干的两个 合并查询marge做出命令改动
git merge origin/master  --allow-unrelated-histories
```

## 25 不同人修改了不同文件如何处理

> **场景：**多人同时维护一个分支的时候 有一个人修改了 一个文件 另一个人修改了不同的文件 这种合并起来 是非常顺利的
>
> **本地与远端的master的关系如下：**
>
> **不是 Fast-Forward：**两个<master>没有共同的祖先
>
> **不是 No-Fast-Forward: ** 不是基于远端的commit做变更的
>
> **Fast-Forward**
>
> 当前分支合并到另一分支时，如果没有分歧解决，就会直接移动文件指针。这个过程叫做fastforward。
>
> 举例来说，开发一直在master分支进行，但忽然有一个新的想法，于是新建了一个develop的分支，并在其上进行一系列提交，完成时，回到 master分支，此时，master分支在创建develop分支之后并未产生任何新的commit。此时的合并就叫fast forward。
>
> 示例：
>
> 1. 新建一个work tree，在master中做几次commit
> 2. 新建develop的branch，然后再做多次commits
>
> 此时的分支流图如下(gitx)：
>
> **正常合并**
>
> (master)$ git merge develop Updating 5999848..7355122 ***Fast-forward\*** c.txt | 1 + d.txt | 1 + 2 files changed, 2 insertions(+), 0 deletions(-) create mode 100644 c.txt create mode 100644 d.txt
>
> 可以看出这是一次fast-forward式的合并，且合并完之后的视图为扁平状，看不出develop分支开发的任何信息。
>
> **使用–no-ff进行合并**
>
> —no-ff (no fast foward)，使得每一次的合并都创建一个新的commit记录。即使这个commit只是fast-foward，用来避免丢失信息。
>
> (master)$ git merge –no-ff develop **Merge made by recursive.** c.txt | 2 +- d.txt | 2 +- 2 files changed, 2 insertions(+), 2 deletions(-)
>
> 可以看出，使用no-ff后，会多生成一个commit 记录，并强制保留develop分支的开发记录（而fast-forward的话则是直接合并，看不出之前Branch的任何记录）。这对于以后代码进行分析特别有用，故有以下最佳实践。
>
> **好的实践**
>
> –no-ff，其作用是：要求git merge即使在fast forward条件下也要产生一个新的merge commit。此处，要求采用–no-ff的方式进行分支合并，其目的在于，希望保持原有“develop branches”整个提交链的完整性。Git – Fast Forward 和 no fast foward

> ahead 1 behind 1 的意思
>
> **ahead** ：本地分支比远端新一个commit
>
> **behind** ：远端比本地多一个commit
>
> 这种问题解决方案
>
> ```
> # 1. 获取远程分支debug的修改 
> git fetch origin debug 
> 
> # 2.合并远程分支debug  
> git merge origin debug 
> 
> # 更新本地分支  
> 3. git pull origin debug 
> ```

![image-20200213190214751](http://ww1.sinaimg.cn/large/006vQ8sXgy1gfiq9h5s8xj30sg08bjw9.jpg?ynotemdtimestamp=1606703976531)

## 26 不同人修改了同文件的不同区域

> 与远端同步
>
> ```bash
> git pull 
> # 与 25一样
> ```

## 27 两个人修改了同文件的同一区域

> git pull 之后自己做处理
>
> 然后commit 然后 push

## 28 同时变更了文件名和文件内容

> git pull 可以做出智能处理
>
> 然后commit 然后 push

## 29 多人修改同一文件名

> git pull 之后自己做处理
>
> 然后commit 然后 push

## 30 Git 继承使用禁忌

### 1. 禁止向集成分支执行push

> ```bash
> # 直接干掉 强制更新
> git push -f
> ```

### 2. 禁止向继承分支执行变更历史的操作

> 等着别人砍你吧

## 31 如何在GitHub搜索自己感兴趣的项目

> `in:readme` **在readme里面搜索**
>
> `stars:>1000` **stars大于1000的**
>
> **Code Option：**
>
> 'dsc' filename：<filename>
>
> **例子：**'public' + 'static' filename:config.java

## 32 开源项目如何保质保量

> pull request 然后 通过 review与checks

## 33怎样选择适合自己团队的工作流

![image-20200214100653733](http://ww1.sinaimg.cn/large/006vQ8sXgy1gfiq9vbmgqj30po0h8419.jpg?ynotemdtimestamp=1606703976531)

![image-20200214100726831](http://ww1.sinaimg.cn/large/006vQ8sXgy1gfiqam8nn1j30po0iawiw.jpg?ynotemdtimestamp=1606703976531)

![image-20200214100736627](http://ww1.sinaimg.cn/large/006vQ8sXgy1gfiqb0ftc6j30sg0i941i.jpg?ynotemdtimestamp=1606703976531)

![image-20200214100752269](http://ww1.sinaimg.cn/large/006vQ8sXgy1gfiqbbma3tj30oz0grq4r.jpg?ynotemdtimestamp=1606703976531)

![image-20200214100810162](http://ww1.sinaimg.cn/large/006vQ8sXgy1gfiqbvphqaj30rj0gmgo2.jpg?ynotemdtimestamp=1606703976531)

## 34 如何挑选和是的分支集成策略

> 查看一些信息

![image-20200214102353830](http://ww1.sinaimg.cn/large/006vQ8sXgy1gfiqc8wuocj30xq0flaar.jpg?ynotemdtimestamp=1606703976531)

> 选择集成方式
>
> 线性方式：选择-->allow rebase merging
>
> 公共集成分支不允许 push -f
>
> 第一种 产生marge commit

![image-20200214102608582](http://ww1.sinaimg.cn/large/006vQ8sXgy1gfiqcjss4xj30t00ak3z0.jpg?ynotemdtimestamp=1606703976531)

![merge commits](http://ww1.sinaimg.cn/large/006vQ8sXgy1gfiqcu6l1vj30gz08rjs5.jpg?ynotemdtimestamp=1606703976531)

> 第二种 将产生的 commit 组合为一个commit 放入 master

> 第三种 将长生的commit 一个一个跳出来放入master

![image-20200214121045925](http://ww1.sinaimg.cn/large/006vQ8sXgy1gfiqd6pgmdj30gj0a70te.jpg?ynotemdtimestamp=1606703976531)

## 35 启用issue跟踪需求和任务

> 利用project看板来管理issue

## 36 项目内部怎么实施code review

![image-20200214123352048](http://ww1.sinaimg.cn/large/006vQ8sXgy1gfiqdi7bjlj30sy0hgmyp.jpg?ynotemdtimestamp=1606703976531)

## 37团队协作时如何做多分支的集成

```bash
# 让远端集成分支 强制回退
# 不建议这样做
git push -f origin < 要回退commit > :<分支名>
git push -f origin 6b231c14 : master
# 将分支回退
git reset --hard origin/s
# 去了解
git rerere
```