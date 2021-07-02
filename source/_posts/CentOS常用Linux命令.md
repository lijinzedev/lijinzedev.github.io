---
title: CentOS常用Linux命令
top: false
cover: false
toc: true
mathjax: true
categories:
  - linux
tags:
  - linux
date: 2020-11-29 15:08:14
password:
summary:
---

# 1 命令行相关

* <kbd>Ctrl + a</kbd> 与<kbd>Ctrl + e</kbd> 分别代表把光标移动到最前和最后

> 小贴士:
>
> `a`代表单词`ahead`

* <kbd>Ctrl + u</kbd> 与<kbd>Ctrl + k</kbd> 分别代表光标处往前和光标处往后删除

  

# 2 命令相关

## 1 进程相关

[linux之 grep "xxx" * | wc -l 命令](https://blog.csdn.net/sinat_27403673/article/details/84670428)

```bash
# 根据进程名找到进程Id停掉历史进程
ps -aux | grep -v greo | grep ${serverName} | awk '{print $2}' | xargs kill -9
# 查看集群数量
ps -ef | grep nacos | grep -v grep | wc -l
# 去除包含grep的进程行 ，避免影响最终数据的正确性 
grep -v grep 
# 统计行数
wc -l
```

## 2 操作系统信息相关

### 1 操作系统

```bash
# 查看操作系统版本
cat /etc/centos-release
# 查看操作系统内核版本
uname -r
# 查看操作系统详情
uname -a
# 查看更多详情
more /etc.*release
# 查看CPU详情
cat /proc/cpuinfo
```

### 2 磁盘

```bash
df -h 
# df -a 显示全部的文件系统
# df -i 显示innode信息
# df -T 显示文件系统类型
# du -s 指定目录大小汇总
# du -h 带计量单位
# du -a 韩文建

```

