---
title: Linux IP命令
top: false
cover: false
toc: true
mathjax: true
categories:
  - linux
tags:
  - linux
date: 2021-07-02 10:07:50
password:
summary:
---

# 一、引言 

> linux的**ip**命令和**ifconfig**类似，但前者功能更强大，并旨在取代后者。使用ip命令，只需一个命令，你就能很轻松地执行一些网络管理任务。ifconfig是net-tools中已被废弃使用的一个命令，许多年前就已经没有维护了。iproute2套件里提供了许多增强功能的命令，ip命令即是其中之一

# 二、简介

```ruby
[root@localhost ~]# ip
Usage: ip [ OPTIONS ] OBJECT { COMMAND | help }
       ip [ -force ] -batch filename
where  OBJECT := { link | addr | addrlabel | route | rule | neigh | ntable |
                   tunnel | tuntap | maddr | mroute | mrule | monitor | xfrm |
                   netns | l2tp | tcp_metrics | token }
       OPTIONS := { -V[ersion] | -s[tatistics] | -d[etails] | -r[esolve] |
                    -f[amily] { inet | inet6 | ipx | dnet | bridge | link } |
                    -4 | -6 | -I | -D | -B | -0 |
                    -l[oops] { maximum-addr-flush-attempts } |
                    -o[neline] | -t[imestamp] | -b[atch] [filename] |
                    -rc[vbuf] [size]}
```

## 1 OPTIONS

OPTIONS：选项。

- -s：显示出该设备的统计数据(statistics)，例如总接受封包数等；

OBJECT：动作对象，就是是可以针对哪些网络设备对象进行动作。

- link：关于设备 (device) 的相关设定，包括 MTU，MAC 地址等。
- addr/address：关于额外的 IP 设定，例如多 IP 的实现等。
- route ：与路由有关的相关设定。

# 三、常用命令

## 查看网络接口信息

> 可以查看是否为静态IP地址

```bash
# 查看接口
ip addr show 
ip addr
ip a
# 解释
[add|del]：
add|del：进行相关参数的增加(add)或删除(del)设定。
show：显示详细信息。
IP 参数 ：主要就是网域的设定，例如 192.168.100.100/24 之类的设定。
[dev 设备名]：IP 参数所要设定的设备，例如eth0, eth1等。
[相关参数]：
broadcast：设定广播位址，如果设定值是 + 表示让系统自动计算；
label：该设备的别名，例如eth0:0；
scope：这个设备的领域，默认global，通常是以下几个大类：
global：允许来自所有来源的连线；
site：仅支持IPv6 ，仅允许本主机的连接；
link：仅允许本设备自我连接；
host：仅允许本主机内部的连接；

```

> valid_lft 7153sec preferred_lft 3553sec 代表动态IP地址
>
> valid_lft forever preferred_lft forever 有效期永久代表静态IP地址

![命令行](https://raw.githubusercontent.com/lijinzedev/picture/main/img/20210702104308.png)

![image-20210702105002235](https://raw.githubusercontent.com/lijinzedev/picture/main/img/20210702105002.png)

```bash
ip -s link show em1
```

> 输出发送接收的信息

![image-20210702105210778](https://raw.githubusercontent.com/lijinzedev/picture/main/img/20210702105210.png)

## 添加、删除IP地址

```shell
ip address add 192.168.1.232 dev ens160
```

```
ip a del 192.168.1.232 dev ens160
```

## 启用、禁用网络接口

```shell
ip link set ens160 down
```

```shell
ip link set ens160 up
```

## 查看路由信息

```bash
# 格式
ip route [add|del] [IP或网域] [via gateway] [dev 设备]
[IP或网域]：可使用192.168.50.0/24之类的网域或者是单纯的 IP 。
[via gateway]：从哪个gateway出去，不一定需要。
[dev 设备名]：所要设定的设备，例如eth0, eth1等。
# 显示详细信息
ip route show
```

# 四、表格

| 需求                       | iproute                                                      | net-tools                                                    |
| :------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 显示所有已连接的网络接口   | ip link show                                                 | ifconfig -a                                                  |
| 激活或停用网络接口         | ip link set down eth1 <br />ip link set up eth1              | ifconfig   eth1 up<br />ifconfig   eth1 down                 |
| 为网络接口分配IPv4地址     | ip addr add 10.0.0.1/24 dev th1                              | ifconfig eth1 10.0.0.1/24                                    |
| 给同一个接口分配多个IP地址 | ip addr add 10.0.0.1/24 broadcast 10.0.0.255 dev eth1        | ifconfig eth1 add 10.0.0.1                                   |
| 移除网络接口的IPv4地址     | ip addr del 10.0.0.1/24 dev eth1                             | ifconfig eth1 add 0                                          |
| 显示网络接口的IPv4地址     | ip addr show dev eth1                                        | ipconfig eth1                                                |
| 查看IP路由表               | ip route show                                                | route -n<br />netstat -rn                                    |
| 添加修改默认路由           | ip route add default via 192.168.1.2 dev eth0<br />ip route replace default via 192.168.1.2 dev eth0 | route add default  gw 192.16.1.2 dev eth0<br />route del default  gw 192.16.1.2 dev eth0<br /> |
| 添加移除静态路由           | ip route add 10.1.2.0/24 via 192.168.1.1 dev eth0<br />ip route del 10.1.2.0/24 | route add  -net 10.1.2.0/24 gw 192.168.1.1 dev eth0<br />route del -net 10.1.2.0/24 |
| 查看套接字统计信息         | ss<br />ss -l                                                | netstat<br />netstat -l                                      |
| 查看ARP表                  | ip neigh                                                     | arp -an                                                      |
| 添加或删除静态arp          | ip neigh add 192.168.1.100 lladdr 00:01:22:c3:5b:ef dev eth0<br />ip neigh del 192.168.1.100 dev etho | arp -s  192.168.1.100 00:01:22:c3:5b:ef<br />arp -d 192.168.1.100 |

