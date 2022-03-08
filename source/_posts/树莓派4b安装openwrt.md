---
title: 树莓派4b与OpenWRT
top: false
cover: false
toc: true
mathjax: true
categories:
  - 树莓派
tags:
  - 树莓派
date: 2022-03-08 09:41:18
password:
summary:
---

# 树莓派安装OpenWRT

## 硬件资料

- 树莓派4B 4G内存
- TPLINK路由器
- 网线一条

## 镜像软件资料

- [OpenWrt-Rpi Repo](https://link.segmentfault.com/?enc=SgF5m0Rtw1aNsQ4lW1H5Fg%3D%3D.yJsuiAm28Hnkn0ITy8nyqT0Gv65H13HUWlbXGfVeDp%2BmYOpc6lQaSA4R2BiTNH0W)
  - 我下载的镜像是`openwrt-bcm27xx-bcm2711-rpi-4-ext4-factory.img`，各个版本的镜像的差别具体可查看Repo中的说明
- [Raspberry官方的sd卡烧录工具](https://link.segmentfault.com/?enc=cbdqOkdAEOxNXtZ8xhM1sQ%3D%3D.1Tv6sEop0nn05knux46ffxrhFA2G5mrQaO3sF1JtMikQwj2GpgU8yP4e3uTwtl6X)

## 安装步骤

### 一. 连接到树莓派

- 将镜像烧录到SD卡，插入树莓派后启动电源，启动后树莓派会发射名称为OpenWrt的无验证的WiFi热点，直接用电脑连接此WiFi，ssh连接树莓派`ssh root@192.168.1.1`，密码为`password`

### 二.初始化网络设置

- 首先确认树莓派的上级网段，确认当前环境的上网设备，比如家庭路由器或者实验室的路由器等，进入其设置界面，查看其局域网网段，并查看网段中已经占用的IP，之后任意选择一个未占用的IP（网段IP以`192.168.3.1`为例，假设IP`192.168.3.250`未被占用）
  - 更简单的方法是，查看自己的上网设备（手机，笔记本）的IP，得到网段IP，之后任选一个比较靠后的IP（网段IP以`192.168.3.1`为例，则可选IP是`192.168.3.x` x为2-254），然后尝试`ping`此IP，或者使用其他方式确定此IP未被占用
- 在树莓派OpenWrt shell环境中执行

- 在树莓派OpenWrt shell环境中执行

```bash
uci set network.lan.ipaddr=192.168.3.250
uci commit network
/etc/init.d/network restart
```

- 更改生效后，重新连接OpenWrt WiFi后，在浏览器进入OpenWrt管理界面，访问`192.168.3.250`，用户名`root`，密码`password`，进入控制面板后，在 “⽹络 - 接⼝ - Lan - 修改” 中进⾏以下设置：

![image-20220308105210570](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202203081052651.png)

![image-20220308105235107](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202203081052139.png)

![image-20220308105249683](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202203081052711.png)

### 三. 部署完毕

- 树莓派断电，使用网线连接家庭路由器或者实验室路由器的LAN 口到树莓派的网口（**在此之前，树莓派不应该使用网线连接到路由器**），树莓派上电
- 等待开启稳定后，尝试连接OpenWrt WiFi，发现可以上网了

### 四. 推荐使用的配置与服务

- OpenWrt系统更改密码
- OpenWrt 控制面板更改密码
- OpenWrt WiFi添加密码验证
- 使用ADGuard Home屏蔽广告
  - [参考](https://link.segmentfault.com/?enc=RrVl0yAe%2BTJk379%2BZ9GAzw%3D%3D.WFyOvoej68WtCfqpemSx4FgFs%2FaArrnpRPTF%2FRh%2F4SU%3D)
- 更多功能等待挖掘使用...

# 参考资料

- [自编译 OpenWrt 系列 - 旁路由设置指南 | 美丽应用](https://mlapp.cn/1008.html) 

# AdGuardHome

1. 更新核心
2. 作为dnsmasq的上游服务器

https://www.right.com.cn/forum/thread-4037690-1-1.html

https://www.bilibili.com/read/cv12276105
