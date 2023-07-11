---
title: 记一次linux服务器磁盘满问题排查
top: false
cover: false
toc: true
mathjax: true
categories:
  - 运维
  - linux
tags:
  - linux
  - 问题排查
date: 2022-06-27 14:11:13
password:
summary:
---

# linux服务器磁盘满了怎么办

## 方法一

#### 步骤一：遇到磁盘空间不足的报错时候，首先使用df -h查看磁盘空间使用情况。如下/ 目录磁盘空间达到100%。

```css
[root@localhost]# df -h
文件系统                    容量  已用  可用 已用% 挂载点
devtmpfs                 4.8G     0  4.8G    0% /dev
tmpfs                    4.8G     0  4.8G    0% /dev/shm
tmpfs                    4.8G  489M  4.3G   11% /run
tmpfs                    4.8G     0  4.8G    0% /sys/fs/cgroup
/dev/mapper/centos-root   50G   46G  4.4G   92% /
/dev/sda1               1014M  151M  864M   15% /boot
/dev/mapper/centos-home   46G  6.9G   39G   16% /home
overlay                   50G   46G  4.4G   92% /var/lib/docker/overlay2/47b554f3873600e28d1a55edf89e8e0283d47ae826344df63b35290169c4890a/merged
tmpfs                    970M     0  970M    0% /run/user/0
```

#### 步骤二：进入目录/，查找磁盘空间中的大文件，使用命令

`du -h --max-depth=1 /`

`du -sh *`查找占用空间大的目录，可以看到tomcat空间占用的空间比较大，通过逐层定位，最后会找到具体的文件

#### 步骤三：除了上面逐层定位的方法，我们也可以直接查找出大文件，使用命令find / -size +400M查找出大于400M的文件

#### 步骤四：从上面可以看出，是/home/tomcat/logs/目录下的日志文件占用空间较大，如果判定日志文件已经无用，直接删除即可保留最近的日志文件，其余删除，空间释放

## 方法二

#### 步骤一：除了磁盘空间除了文件占用之外，还有一种情况，当磁盘空间满了之后，我们无法查找到大文件，此时可能是文件可能已经被删掉，但有进程依然在使用它。在进程运行期间，Linux 不会释放该文件的存储空间。此时看到磁盘空间仍是100%

![在这里插入图片描述](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202206271418110.png)

#### 步骤二：此时适用命令lsof | lsof | grep deleted 查找到占用的进程，直接停止进程或者kill掉就可以释放空间（注：如果不是生产环境，重启操作系统，空间也会释放）

![在这里插入图片描述](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202206271418189.png)

## 方法三

最后一种情况，就是随着linux系统应用的安装，当初磁盘空间申请过小，磁盘使用达到100%，也没有可以清理的磁盘空间，此时如果其余磁盘[挂载](https://so.csdn.net/so/search?q=挂载&spm=1001.2101.3001.7020)点有充足的空间，我们可以通过软连接使用其余磁盘的空间，或者将应用安装到富余的目录空间，此外，现在很多磁盘都使用LVM逻辑卷的方式挂载，增加磁盘后，可以使用`动态扩容`磁盘空间解决。

# docker json.log 日志太大

服务器上磁盘空间不足了，看了一下原来是某个docker进程下的json.log文件占了140G。

网上搜了一下原来docker默认情况下会将程序的stdout写入到 **[CONTAINER ID]-json.log** 文件中。

解决方法是：#添加max-size参数来限制文件大小docker run -it --log-opt max-size=10m --log-opt max-file=3 alpine ash

临时清理办法 cat /dev/null > 容器id-json.log 通过rm -rf或者文件管理器删除文件，将会从文件系统的目录结构上解除链接（unlink）。如果文件是被打开的（有一个进程正在使用），那么进程将仍然可以读取该文件，磁盘空间也一直被占用。正确姿势是cat /dev/null > *-json.log，当然你也可以通过rm -rf删除后重启docker。

参考文档：[https://docs.docker.com/config/containers/logging/json-file/](https://link.jianshu.com/?t=https%3A%2F%2Fdocs.docker.com%2Fconfig%2Fcontainers%2Flogging%2Fjson-file%2F)
