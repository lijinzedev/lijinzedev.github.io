---
title: mac man汉化
top: false
cover: false
toc: true
mathjax: true
categories:
  - Mac
  - 环境搭建
tags:
  - Mac
date: 2021-09-14 14:19:43
password:
summary:
---

# 记录mac man汉化过程

## 一、环境

* 操作系统：MacBook Pro (13-inch, M1, 2020)

系统版本：Big Sur 11.5.2

## 二、汉化过程

* man数据目录位置

```bash
/usr/share/man/ ls
man1 man4 man5 man6 man7 man8 man9 mann
```

* 修改man.conf

```bash
sudo vim /etc/man.conf
```

* 找到MANPATH，自行添加

```bash
# 例如
MANPATH /Users/curiosity/man/LinuxManCN
```

* 两条命令

```text
man -aw
man -M
```

`-aw`选项可以让你查看你的机器上到底有多少种不同版本的man

`-M`选项可以让我们指定某个版本的manpage

```bash
man -M /Users/curiosity/man/LinuxManCN mkdir
```

就可以查看Linux的中文版mkdir了。

如果想灵活地在多个doc之间切换，可以尝试在.zshrc里面添`alias`，而不去配置MANPATH。

```bash
alias linuxmancn='man -M $HOME/Documents/LinuxManCN'
```

![配置](ttps://raw.githubusercontent.com/lijinzedev/picture/main/img/202109141504895.png)

**执行**

```bash
source ~/.zshrc
```

## 三、解决乱码

* 安装 groff

m1的brew默认安装在/opt下面

```bash
brew install groff
```

注意：

> 这条命令在我这里并不能软连接到/usr/local/bin我使用了绝对路径

```text
brew link groff
```

* 配置man.conf

**添加**

```ruby
NROFF preconv -e utf8 | /usr/local/bin/groff -Wall -mtty-char -Tutf8 -mandoc -c
PAGER /usr/bin/less -isR
```

## 参考

1. [^](https://zhuanlan.zhihu.com/p/349918075#ref_1_0)Linux英文man下载 https://mirrors.edge.kernel.org/pub/linux/docs/man-pages/
2. [^](https://zhuanlan.zhihu.com/p/349918075#ref_2_0)Linux中文man下载 https://github.com/man-pages-zh/manpages-zh/tree/master/src
3. [^](https://zhuanlan.zhihu.com/p/349918075#ref_3_0)m1的macbook安装arm的brew https://blog.csdn.net/u013474815/article/details/113154668
4. [^](https://zhuanlan.zhihu.com/p/349918075#ref_4_0)Github-Issue https://github.com/man-pages-zh/manpages-zh/issues/22
5. [^](https://zhuanlan.zhihu.com/p/349918075#ref_4_0)大佬提出的m1 解决方案https://zhuanlan.zhihu.com/p/349918075

