---
title: CentOS配置网络
top: false
cover: false
toc: true
mathjax: true
categories:
  - linux - CentOS
tags:
  - linux - CentOS
date: 2020-11-29 15:46:24
password:
summary:
---

# CentOS配置网络

```bash
# 命令
vi /etc/sysconfig/network-scripts/ifcfg-ens33

TYPE=Ethernet                # 网卡类型：为以太网
PROXY_METHOD=none            # 代理方式：关闭状态
BROWSER_ONLY=no                # 只是浏览器：否
# BOOTPROTO=dhcp  # 网卡的引导协议：DHCP[中文名称: 动态主机配置协议]
BOOTPROTO=static
IPADDR=192.168.0.110 # 静态ip
GATEWAY=192.168.0.1 # 网关
NETMASK=255.255.255.0 # 子网掩码
DNS1=192.168.1.1 # dns
DEFROUTE=yes                # 默认路由：是, 不明白的可以百度关键词 `默认路由` 
IPV4_FAILURE_FATAL=no        # 是不开启IPV4致命错误检测：否
IPV6INIT=yes                # IPV6是否自动初始化: 是[不会有任何影响, 现在还没用到IPV6]
IPV6_AUTOCONF=yes            # IPV6是否自动配置：是[不会有任何影响, 现在还没用到IPV6]
IPV6_DEFROUTE=yes            # IPV6是否可以为默认路由：是[不会有任何影响, 现在还没用到IPV6]
IPV6_FAILURE_FATAL=no        # 是不开启IPV6致命错误检测：否
IPV6_ADDR_GEN_MODE=stable-privacy            # IPV6地址生成模型：stable-privacy [这只一种生成IPV6的策略]
NAME=ens33                    # 网卡物理设备名称
UUID=f47bde51-fa78-4f79-b68f-d5dd90cfc698    # 通用唯一识别码, 每一个网卡都会有, 不能重复, 否两台linux只有一台网卡可用
DEVICE=ens33                    # 网卡设备名称, 必须和 `NAME` 值一样
ONBOOT=yes  # 设置网卡启动方式为 开机启动 并且可以通过系统服务管理器 systemctl 控制网卡   


```

```bash
# 重启网络
service network restart
systemctl restart network
```

