---
title: CFW配置
top: false
cover: false
toc: true
mathjax: true
categories:
  - cfw
tags:
  - cfw
date: 2021-04-24 11:29:00
password:
summary:
---

# CFW一些概念

## 系统代理

​		`系统代理设置`顾名思义就是在`系统设置`里面设置一个`代理服务器`，让软件可以直接调用`系统代理设置`直接连接`代理服务器`，而不需要单独的配置。这样所有的软件都可以知道现在有一个`代理服务器`可以连接，而且只要跟随`系统代理设置`即可连接，无需额外配置。

​		一般而言，只有浏览器（包括内嵌在各种软件中的浏览器，比如 WeGame、优酷、迅雷9等软件中的内嵌浏览器）会自动调用`系统代理`进行连接。而其它大部分应用一般是不会自动启用`系统代理`进行连接的，要在支持使用代理的软件里面手动设置， **所以这个选项的设置不会影响到这些软件** 。

这是因为绝大部分需要进行代理的需求都在浏览器上，其它软件很少有这个需要（如果有的话一般会提供配置和开关给用户）。

查看系统代理设置：

- Win 10：设置 > 网络和 Internet > 代理
- Win 7：[windows 7 系统如何设置代理服务器_百度经验](https://jingyan.baidu.com/article/0aa22375866c8988cc0d648c.html)

所以，`系统代理设置`控制的就是这个，它有三个选项：

###  直连模式

`直接模式`会在`系统代理设置`里关闭代理，使启用`系统代理设置`的软件（一般为浏览器）直接连接网络。

但是，它并没有关闭在本地构建的`代理服务器`，其它手动配置代理的软件仍然可以进行连接。

###  PAC模式

`PAC模式`会在`系统代理设置`设置一个`PAC脚本`文件，让系统通过这个文件自动选择每一个连接是否启用`代理服务器`，以及选择哪一个`代理服务器`。

![脚本](image-20210424135543038.png)

### 全局模式

全局模式代表作为软件本身接管的所有流量转发到代理服务器，有些软件跑在系统层，甚至不遵循代理规则 ，不把通讯流量交给代理软件接管，这种情况全局代理失效，比如应用商店本身，UWP应用，quest2vr，UDP游戏等

`全局模式`会在`系统代理设置`手动设置一个`代理服务器`，所有跟随`系统代理设置`的软件（一般是浏览器）都会使用这个`代理服务器`。

![全局模式](image-20210424135603937.png)

### 代理流程图

![代理流程图](image-20210424135718829.png)

# TUN模式

## 简介

​		对于不遵循系统代理的软件，TUN 模式可以接管其流量并交由 CFW 处理，在 Windows 中，TUN 模式性能比 TAP 模式好。

​		Tun 模式可以通过新建一个  Tun 虚拟网卡接受操作系统的三层流量，从而拓展 Clash 入口（inbound) 转发能力。Tun 模式有以下潜在的优点：

- 提升 Clash 处理 UDP 的能力
- 从Inbound发回三层流量时，IP 源地址可由 Clash 控制
  - 因此在使用 socks5或shadowsoks协议时，可以表达 socks5/ss 协议发回的 UDP 流量中不同的源IP地址
  - 因此，有可能通过 OutBound 代理实现 STUN
  - 因此，对在代理条件下很多游戏的体验会有提升
- 可以劫持任何三层流量，Clash 可以在任何IP地址和任何端口提供某些服务，非常灵活
  - 因而可以实现 DNS 劫持
- 可以与操作系统的网络栈结合，利用 iptables 等组件的能力

## 安装

### Windows

启动 TUN 模式需要进行如下操作：

1. 进入网站[Wintun (opens new window)](https://www.wintun.net/)，点击界面中`Download Wintun xxx`下载压缩包，根据系统版本将对应目录中`wintun.dll`复制至`Home Directory`目录中。基于`x64`的处理器的`64`位操作系统请使用`amd64`版本，`32`位操作系统请使用`x86`版本
2. 点击`General`中`Service Mode`右边`Manage`，在打开窗口中安装服务模式，安装完成应用会自动重启，Service Mode 右边地球图标变为`绿色`即安装成功
3. 在使用的配置文件中加入如下内容：

```yaml
dns:
  enable: true
  enhanced-mode: redir-host
  nameserver:
    - 8.8.8.8 # 真实请求DNS，可多设置几个
    - 114.114.114.114
# interface-name: WLAN # 出口网卡名称，或者使用下方的自动检测
tun:
  enable: true
  stack: gvisor
  dns-hijack:
    - 198.18.0.2:53
  macOS-auto-route: true
  macOS-auto-detect-interface: true # 自动检测出口网卡
```

### 注意事项

当`enhanced-mode`设置为`fake-ip`时，会出现系统检测到网卡无法联网，微软系 APP 无法登陆使用等问题，可以通过添加`fake-ip-filter`解决：

```yaml
dns:
  enable: true
  enhanced-mode: fake-ip
  nameserver:
    - 114.114.114.114
  fake-ip-filter:
    - "dns.msftncsi.com"
    - "www.msftncsi.com"
    - "www.msftconnecttest.com"
```

**TIP**

> TUN 模式更推荐使用 redir-host 模式

### 使用

> 打开Minin即可

![使用](image-20210424135213647.png)



# TAP模式

对于不遵循系统代理的软件，TAP 模式可以接管其流量并交由 CFW 处理

对于 0.13.8 及以后版本，更推荐使用[TUN 模式](https://docs.cfw.lbyczf.com/contents/tun.html)

## 安装 TAP 网卡

点击`General`页面中`TAP Device`选项的`Manage`按钮，在弹出对话框中选择`Install`将会安装 TAP 网卡，此网卡用于接管系统流量，安装完成可在系统网络连接中看到名为`cfw-tap`的网卡

## 启动 TAP 模式

使用的 Profile 中包含 listen 设置：

```yaml
dns:
  enable: true
  enhanced-mode: redir-host # 或 fake-ip
  listen: 0.0.0.0:53
  nameserver:
    - 223.5.5.5
```

## 工作原理

此版本可以通过设置 Interface Name (自动识别) 属性避免回环，并且支持了 UDP 及 IP 类请求，请在`Settings`页面`Interface Name`选项中选择出站网卡（通常为本机物理网卡）

## 注意事项

当`enhanced-mode`设置为`fake-ip`时，会出现系统检测到网卡无法联网，微软系 APP 无法登陆使用等问题，可以通过添加`fake-ip-filter`解决：

```yaml
dns:
  enable: true
  enhanced-mode: fake-ip
  listen: 0.0.0.0:53
  nameserver:
    - 223.5.5.5
  fake-ip-filter:
    - "dns.msftncsi.com"
    - "www.msftncsi.com"
    - "www.msftconnecttest.com"
```

TIP

> TAP 模式更推荐使用 redir-host 模式

# TUN与TAP模式局域网共享网络

* 首先打开网络适配器`控制面板->网络和Internet->网络连接`
* 笔记本打开移动热点

* TUN选择Clash

![TUN虚拟网卡](image-20210424141013157.png)

* 右键属性点击共享，勾选允许其他用户，家庭网络连接选择本地连接（即wifi）

![属性](image-20210424141059391.png)

* TAP选择TAP-Windows Adapter V9

![TAP虚拟网卡](image-20210424141303539.png)

* 同TUN配置

# Docker安装openwrt

- 给Docker创建网络，运行以下命令，其中`192.168.1.0`和`192.168.1.1`都要修改到自己的对应网段`192.168.x.0`和`192.168.x.1`

```shell
docker network create -d bridge --subnet=192.168.173.0/24 --gateway=192.168.173.1 -o parent=eth0 macnet
```

- 拉取openwrt镜像，这里我使用的是buddyfly的openwrt-aarch64，在docker hub找到的docker镜像，地址：[buddyfly/openwrt-aarch64](https://hub.docker.com/r/buddyfly/openwrt-aarch64) ,以下配置基于这个docker镜像。

```shell
docker pull sulinggg/openwrt:latest
```
* 开启openwrt容器，并把这个容器命名为openwrt

```shell
docker run --restart always --name openwrt -d --network macnet --privileged sulinggg/openwrt:latest /sbin/init
```

* 修改docker内的openwrt网络设置，首先输


```shell
docker exec -it openwrt bash
```

* 输入之后会进入docker容器内，修改网络设置`vi /etc/config/network`

  输入i开始编辑，编辑完成之后按ESC退出编辑，然后输入:wq（不要忘记冒号）回车保存，这里需要给容器内的openwrt指定静态地址，网关和dns，这些配置要和你家中的网络处于一个网段。

![网络配置](image-20210424171346205.png)

* 重启docker

```shell
docker restart openwrt
```

> 在wsl2环境下无法方法，之后树莓派修好了试一下

**参考文章**

[树莓派安装openwrt](https://mlapp.cn/376.html)