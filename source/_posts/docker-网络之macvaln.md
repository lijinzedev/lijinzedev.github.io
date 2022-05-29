---
title: docker 网络之macvaln
top: false
cover: false
toc: true
mathjax: true
categories:
  - 运维
  - docker
tags:
  - 运维
  - docker
date: 2021-07-14 22:10:38
password:
summary:
---

# 一 docker macvlan网络配置

> 在 Docker 中，macvlan 是众多 Docker 网络模型中的一种，并且是一种跨主机的网络模型，作为一种驱动（driver）启用（-d 参数指定），Docker macvlan 只支持 bridge 模式。

## 1 环境搭建

- 主机上配置的eth0网口或者创建的vlan网口,均需要开启混杂模式,命令 `ip link set eth0 promisc on` `ip link set eth0.100 promisc on`

**注意** : 如果不开启混杂模式,会导致macvlan网络无法访问外界,具体在不使用vlan时,表现为无法ping通路由,无法ping通同一网络内其他主机

### 1.1 网络设备

`ip link` 命令用于查询或者设置网络设备。

### 1.2 查看网卡信息

```
root@ubuntu:/home/docker/mysql/config# ip link show eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether dc:a6:32:3a:85:d6 brd ff:ff:ff:ff:ff:ff
```

### 1.3 启用禁用

```
$ ip link set enp0s8 up
$ ip link set enp0s8 down
```

### 1.4 网卡混杂模式

正常模式下，网卡将过滤目的 [MAC地址](https://network.fasionchan.com/zh_CN/latest/protocols/ethernet.html#mac-address) 不是自己的数据包。 在某些场景，比如网络嗅探，我们需要抓取并分析其他网络数据包。 这时，可以为网卡开启 [混杂模式](https://network.fasionchan.com/zh_CN/latest/protocols/ethernet.html#promisc-mode) 。 该模式开启后后，网卡将接受到达接口的所有数据包，不管 [MAC地址](https://network.fasionchan.com/zh_CN/latest/protocols/ethernet.html#mac-address) 是啥。

运行以下命令，为网卡 `eth0` 开启混杂模式：

```
root@ubuntu:~$ sudo ip link set eth0 promisc on
```

操作完毕后，再次查询网卡状态，将看到 `PROMISC` 标识：

```ruby
rootn@ubuntu:~$ ip link show enp0s8
2: enp0s8: <BROADCAST,MULTICAST,PROMISC,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether dc:a6:32:3a:85:d6 brd ff:ff:ff:ff:ff:ff
```

### 1.5 创建docker macvlan网络

```shell
docker network create -d macvlan --subnet=192.168.173.0/24 --gateway=192.168.173.1 --ip-range=192.168.173.197/27 -o parent=eth0 -o macvlan_mode=bridge mac
# 创建容器
docker run -itd --name d2  --network mac busybox
```

查看容器IP

```bash
docker inspect --format='{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -aq)
```

# 参阅

> 主要文章参阅,基本知识都在,后续深入学习

[网卡混杂模式](https://network.fasionchan.com/zh_CN/latest/toolkit/ip.html)
[Docker macvlan网络模式下容器与宿主机互通](https://rehtt.com/index.php/archives/236/)

[Docker 网络模型之 macvlan 详解，图解，实验完整](https://www.cnblogs.com/bakari/p/10893589.html)

[macvlan](https://blog.oddbit.com/post/2018-03-12-using-docker-macvlan-networks/)
