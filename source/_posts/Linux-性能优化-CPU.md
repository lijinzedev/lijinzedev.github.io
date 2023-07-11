---
title: Linux 性能优化-CPU
top: false
cover: false
toc: true
mathjax: true
categories:
  - linux
  - 性能优化
tags:
  - 性能优化
date: 2023-07-11 23:09:22
password:
summary:
---

# 1 环境准备

```bash

docker pull  ubuntu:18.04 # 准备镜像
docker run -i -t --name ubuntuTest ubuntu bash # 进入容器
cat /etc/issue  # 查看ubuntu系统版本
docker start -i ubuntuTest  # -i启动容器，可以进入终端交互。

vim /etc/apt/sources.list  # 更新软件源
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse

apt-get update  # 更新软件源信息
apt-get install vim  # 安装 vim
apt-get install git python3  # 安装 git 和 python3

```

# 2 平均负载

> 平均负载是指单位时间内，系统处于**可运行状态**和**不可中断状态**的平均进程数，也就是**平均活跃进程数**，它和 CPU 使用率并没有直接关系
>
> 可运行状态的进程，是指正在使用 CPU 或者正在等待 CPU 的进程，也就是我们常用 ps 命令看到的，处于 R 状态（Running 或 Runnable）的进程。
>
> 不可中断状态的进程则是正处于内核态关键流程中的进程，并且这些流程是不可打断的，比如最常见的是等待硬件设备的 I/O 响应，也就是我们在 ps 命令中看到的 D 状态（Uninterruptible Sleep，也称为 Disk Sleep）的进程。
>
> 因此，可以简单理解为，平均负载其实就是平均活跃进程数。平均活跃进程数，直观上的理解就是单位时间内的活跃进程数，但它实际上是活跃进程数的指数衰减平均值。这个“指数衰减平均”的详细含义你不用计较，这只是系统的一种更快速的计算方式，你把它直接当成活跃进程数的平均值也没问题。



## 2.1 平均负载为多少时合理

我们知道，平均负载最理想的情况是等于 CPU 个数。所以在评判平均负载时，**首先你要知道系统有几个 CPU**，这可以通过 top 命令或者从文件 /proc/cpuinfo 中读取，比如：

```bash
grep 'model name' /proc/cpuinfo | wc -l # 查看cpu核数
```

有了 CPU 个数，我们就可以判断出，当平均负载比 CPU 个数还大的时候，系统已经出现了过载。

在实际生产环境中，平均负载多高时，需要我们重点关注呢？

**当平均负载高于 CPU 数量 70% 的时候**，你就应该分析排查负载高的问题了。一旦负载过高，就可能导致进程响应变慢，进而影响服务的正常功能。

但 70% 这个数字并不是绝对的，最推荐的方法，还是把系统的平均负载监控起来，然后根据更多的历史数据，判断负载的变化趋势。当发现负载有明显升高趋势时，比如说负载翻倍了，你再去做分析和调查。

## 2.2 平均负载与 CPU 使用率

，平均负载是指单位时间内，处于可运行状态和不可中断状态的进程数。所以，它不仅包括了**正在使用 CPU** 的进程，还包括**等待 CPU** 和**等待 I/O** 的进程。

而 CPU 使用率，是单位时间内 CPU 繁忙情况的统计，跟平均负载并不一定完全对应。比如：

* CPU 密集型进程，使用大量 CPU 会导致平均负载升高，此时这两者是一致的；
* I/O 密集型进程，等待 I/O 也会导致平均负载升高，但 CPU 使用率不一定很高；
* 大量等待 CPU 的进程调度也会导致平均负载升高，此时的 CPU 使用率也会比较高。

## 2.3 平均负载案例分析

下面，我们以三个示例分别来看这三种情况，并用 iostat、mpstat、pidstat 等工具，找出平均负载升高的根源。

```bash
apt-get install stress  # 安装 stress
apt-get install sysstat  # 安装 sysstat
```

stress 是一个 Linux 系统压力测试工具，这里我们用作异常进程模拟平均负载升高的场景。

sysstat 包含了常用的 Linux 性能工具，用来监控和分析系统的性能。案例会用到这个包的两个命令 mpstat 和 pidstat。

* mpstat 是一个常用的多核 CPU 性能分析工具，用来实时查看每个 CPU 的性能指标，以及所有 CPU 的平均指标。
* pidstat 是一个常用的进程性能分析工具，用来实时查看进程的 CPU、内存、I/O 以及上下文切换等性能指标。

```bash
uptime  # 查看系统负载情况
```

### 场景一：CPU 密集型进程

首先，我们在第一个终端运行 stress 命令，模拟一个 CPU 使用率 100% 的场景：

```bash
 stress --cpu 1 --timeout 600

```

接着，在第二个终端运行 uptime 查看平均负载的变化情况：

```bash
# -d 参数表示高亮显示变化的区域
$ watch -d uptime
...,  load average: 1.00, 0.75, 0.39
```

最后，在第三个终端运行 mpstat 查看 CPU 使用率的变化情况：

```bash
# -P ALL 表示监控所有 CPU，后面数字 5 表示间隔 5 秒后输出一组数据
$ mpstat -P ALL 5
Linux 4.15.0 (ubuntu) 09/22/18 _x86_64_ (2 CPU)
13:30:06     CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
13:30:11     all   50.05    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00   49.95
13:30:11       0    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
13:30:11       1  100.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00
```

从终端二中可以看到，1 分钟的平均负载会慢慢增加到 1.00，而从终端三中还可以看到，正好有一个 CPU 的使用率为 100%，但它的 iowait 只有 0。这说明，平均负载的升高正是由于 CPU 使用率为 100% 。

那么，到底是哪个进程导致了 CPU 使用率为 100% 呢？你可以使用 pidstat 来查询：

```
# 间隔 5 秒后输出一组数据
$ pidstat -u 5 1
13:37:07      UID       PID    %usr %system  %guest   %wait    %CPU   CPU  Command
13:37:12        0      2962  100.00    0.00    0.00    0.00  100.00     1  stress
```

### 场景二：I/O 密集型进程

首先还是运行 stress 命令，但这次模拟 I/O 压力，即不停地执行 sync：

```bash
$ stress -i 1 --timeout 600

```

还是在第二个终端运行 uptime 查看平均负载的变化情况：

```bash
$ watch -d uptime
...,  load average: 1.06, 0.58, 0.37
```

然后，第三个终端运行 mpstat 查看 CPU 使用率的变化情况：

```
# 显示所有 CPU 的指标，并在间隔 5 秒输出一组数据
$ mpstat -P ALL 5 1
Linux 4.15.0 (ubuntu)     09/22/18     _x86_64_    (2 CPU)
13:41:28     CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
13:41:33     all    0.21    0.00   12.07   32.67    0.00    0.21    0.00    0.00    0.00   54.84
13:41:33       0    0.43    0.00   23.87   67.53    0.00    0.43    0.00    0.00    0.00    7.74
13:41:33       1    0.00    0.00    0.81    0.20    0.00    0.00    0.00    0.00    0.00   98.99
```

从这里可以看到，1 分钟的平均负载会慢慢增加到 1.06，其中一个 CPU 的系统 CPU 使用率升高到了 23.87，而 iowait 高达 67.53%。这说明，平均负载的升高是由于 iowait 的升高。

那么到底是哪个进程，导致 iowait 这么高呢？我们还是用 pidstat 来查询：

```
# 间隔 5 秒后输出一组数据，-u 表示 CPU 指标
$ pidstat -u 5 1
Linux 4.15.0 (ubuntu)     09/22/18     _x86_64_    (2 CPU)
13:42:08      UID       PID    %usr %system  %guest   %wait    %CPU   CPU  Command
13:42:13        0       104    0.00    3.39    0.00    0.00    3.39     1  kworker/1:1H
13:42:13        0       109    0.00    0.40    0.00    0.00    0.40     0  kworker/0:1H
13:42:13        0      2997    2.00   35.53    0.00    3.99   37.52     1  stress
13:42:13        0      3057    0.00    0.40    0.00    0.00    0.40     0  pidstat
```

### 场景三：大量进程的场景

当系统中运行进程超出 CPU 运行能力时，就会出现等待 CPU 的进程。

比如，我们还是使用 stress，但这次模拟的是 8 个进程：

```bash 
stress -c 8 --timeout 600
```

由于系统只有 2 个 CPU，明显比 8 个进程要少得多，因而，系统的 CPU 处于严重过载状态，平均负载高达 7.97：

```bash
$ uptime
...,  load average: 7.97, 5.93, 3.02

```

接着再运行 pidstat 来看一下进程的情况：

```
# 间隔 5 秒后输出一组数据
$ pidstat -u 5 1
14:23:25      UID       PID    %usr %system  %guest   %wait    %CPU   CPU  Command
14:23:30        0      3190   25.00    0.00    0.00   74.80   25.00     0  stress
14:23:30        0      3191   25.00    0.00    0.00   75.20   25.00     0  stress
14:23:30        0      3192   25.00    0.00    0.00   74.80   25.00     1  stress
14:23:30        0      3193   25.00    0.00    0.00   75.00   25.00     1  stress
14:23:30        0      3194   24.80    0.00    0.00   74.60   24.80     0  stress
14:23:30        0      3195   24.80    0.00    0.00   75.00   24.80     0  stress
14:23:30        0      3196   24.80    0.00    0.00   74.60   24.80     1  stress
14:23:30        0      3197   24.80    0.00    0.00   74.80   24.80     1  stress
14:23:30        0      3200    0.00    0.20    0.00    0.20    0.20     0  pidstat
```

可以看出，8 个进程在争抢 2 个 CPU，每个进程等待 CPU 的时间（也就是代码块中的 %wait 列）高达 75%。这些超出 CPU 计算能力的进程，最终导致 CPU 过载。

## 小结

分析完这三个案例，我再来归纳一下平均负载的理解。

平均负载提供了一个快速查看系统整体性能的手段，反映了整体的负载情况。但只看平均负载本身，我们并不能直接发现，到底是哪里出现了瓶颈。所以，在理解平均负载时，也要注意：

* 平均负载高有可能是 CPU 密集型进程导致的；
* 平均负载高并不一定代表 CPU 使用率高，还有可能是 I/O 更繁忙了；
* 当发现负载高的时候，你可以使用 mpstat、pidstat 等工具，辅助分析负载的来源。
