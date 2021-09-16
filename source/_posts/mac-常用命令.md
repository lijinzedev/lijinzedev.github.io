---
2title: mac 常用命令
top: false
cover: false
toc: true
mathjax: true
categories:
  - mac	
tags:
  - mac
date: 2021-09-16 10:26:20
password:
summary:
---

# 一、进程管理

## 1.1 PS

| 命令                                 | 解释                       |
| ------------------------------------ | -------------------------- |
| `ps -A` <br />`ps -e` <br />`ps -ax` | 查看系统所有的进程信息     |
| `ps -ef `                            | 查看详细的信息             |
| `id-nu <num>`                        | 查看UID对应的人信息        |
| `ps -p <pid >`                       | 查看pid详细信息            |
| `ps -u <username>`                   | 查看username对应的进程信息 |
| `ps -U <uid>`                        | 查看UID对应的进程信息      |
| `ps -A -o user -o pid  -o comm`      | 指定输出信息               |
| `ps -L `                             | 查询所有的关键词           |

## 1.2 kill

| 命令            | 解释     |
| --------------- | -------- |
| `kill -9 <pid>` | 杀掉进程 |

## 1.3 killall

> 批量的结束进程通过进程名

| 命令                | 解释     |
| ------------------- | -------- |
| `killall -9 <name>` | 杀掉进程 |

## 1.4 plill

> 批量的结束进程通过进程名

| 命令               | 解释     |
| ------------------ | -------- |
| `pkill -9 <name*>` | 杀掉进程 |

# 二、温度、风扇、转速

## 2.1 powermetrics

| 命令                          | 解释           |
| ----------------------------- | -------------- |
| `sudo powermetrics`           | 信息查询       |
| `sudo powermetrics -h`        |                |
| `sudo powermetrics -s <disk>` | 指定采样器名称 |

# 三、定时关机、重启、睡眠

## 3.1 shutdown

`shutdown [-h | -s | -r ] time [提示信息]`

*  -h 立刻关机
* -s 立刻睡眠
* -r 代表重启

**time** 

* now
* +分钟数
* hh:mm今天的几时几分（24小时制） 
* yymmddhhmm指定年份月份日期小时分钟 

`shutdown -s +10`

> 十分钟之后关机
>
> 使用ps - A ｜ grep shutdown 查询
>
> 使用kill 杀掉

## 3.2 reboot

> 立即重启

## 3.3 halt

> 立即关机
>
> = shutdown -h now

# 四、网络命令

## 4.1 ifconfig

![image-20210916134646398](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202109161346164.png)

| 命令                        | 解释             |
| --------------------------- | ---------------- |
| `ifconfig <interface-name>` | 查询指定网卡信息 |

## 4.2 networksetup

```bash
# 查看网络接口简写信息
networksetup -listallhardwareports
```

## 4.3 ipconfig

> 获取ID地址

`ipconfig getifaddr interface-name`

> 获取网关地址

`ipconfig getoption en0 router`

## 4.4 netstat

> 查看网路连接、端口、协议

![image-20210916142724760](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202109161427793.png)

* `-a` 现实所有的连接信息，包括常用于服务器的一些端口监听连接

> 不加-a的话，默认不显示LISTEN连接信息的

* `-n` 不显示别名信息，用数字代替，可以加快命令的执行速度

> 比如：mysql的端口用3306直接显示，而不是用mysql这样的名称显示

* `-p` protocol 显示指定网络协议的连接，全部协议在/etc/protocols中

> 比如：-p tcp 或者 -p udp 也可以是 -p TCP 或者 -p UDP

* `-v `显示更多的信息，可以显示对应连接的进程ID（PID）等

> 当需要查看网络连接是属于哪个进程时可以使用

* `-r` 显示路由表信息
* `-L `显示监听队列的信息
* `-l` 显示出完整的IPV6的地址信息

## 4.5 lsof

> 查看系统打开文件信息比netstat更加友好

* `-i` 显示所有打开的网路连接

> -i4 => ipv4 连接
>
> -i6 => ipv6 连接
>
> -iTCP =>  tcp 连接
>
> -iTCP:8080 => tcp8080 端口连接 

* `-s` 与`-i`配合使用，用于指定特定的协议和特定的网络状态

> lsof -iTCP -sTCP:LISTEN

* `-n` 不显示别名信息，用数字代替，可以加快命令的执行速度

> 比如：mysql的端口用3306直接显示，而不是用mysql这样的名称显示

* `-P` 不让端口号与端口名称之间转换，加快命令的执行速度

> -n -P一起使用，-nP可以大大加快的命令的执行速度

**例：**

`sudo lsof -iTCP:3306 -sTCP:LISTEN -nP` 

**使用场景**

> 查看监听状态的端口

```bash
netstat -an | grep LISTEN
netstat -anv | grep -i listen
```

```bash
lsof -iTCP -sTCP:LISTEN -nP
sudo lsof -iTCP -sTCP:LISTEN -nP
```

> 查看指定端口占用情况

```bash
netstat -an | grep 3306
netstat -anv | grep 3306
```

```bash
lsof -iTCP:3306 -sTCP:LISTEN -nP
sudo lsof -iTCP:3306 -sTCP:LISTEN -nP
```

##  4.6 traceroute

> 追踪路由信息

`traceroute [域名|IP地址]`

