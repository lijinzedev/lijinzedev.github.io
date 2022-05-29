---
title: Windows 配置双网关 双Ip
top: false
cover: false
toc: true
mathjax: true
categories:
  - 工具
tags:
  - 工具
date: 2021-06-29 10:40:42
password:
summary:
---

# 配置

1. 打开控制面板-> 网络和 Internet-> 网络连接
2. 手动设置外网IP、网关、DNS服务等



![外网自定义设置](https://raw.githubusercontent.com/lijinzedev/picture/main/img/20210629105148.png)

3. 点击高级手动添加内网IP、子网、网关

![内网设置](https://raw.githubusercontent.com/lijinzedev/picture/main/img/20210629105411.png)

4. 管理员权限打开CMD控制台

5. 添加路由转发

> 配置 172.0.0.0 的网域通过172.16.7.1转发

```bash
route add -p 172.0.0.0 mask 255.0.0.0 172.16.7.1
```

# windows 路由命令

### 1. route命令的基本用法

```shell
ROUTE [-f] [-p] [-4|-6] command [destination] [MASK mask] [gateway] [METRIC metric] [IF interface]
```

| 参数          | 含义                                                         |
| ------------- | ------------------------------------------------------------ |
| [-f]          | 清除所有网关项的路由表。这个参数慎用。                       |
| [-p]          | 增加永久路由。在默认情况下，重启系统之后，<br />我们用add命令增加的路由是不会被保存的，-p参数和add命令结合使用的时候，<br />可以增加永久保存路由。<br />永久路由保存在注册表的这个位置：HKEY_LOCAL_MACH/SYSTEM/CurrentControlSet/Services/Tcpip/Parameters/PersistentRoutescommand。 |
| [-4]          | IPv4网络                                                     |
| [-6]          | IPv6网络                                                     |
| [command]     | 共有4个命令：print, add, delete, change                      |
| [destination] | 目标地址，结合MASK，可以定义主机或者网段。                   |
| [mask]        | 定义子网掩码，如果没有定义mask，默认为255.255.255.255，说明destination是一台主机，而不是一个网段 |
| [gateway]     | 定义网关的地址，就是数据的下一跳地址。如果不指定，系统会查找最佳的网关 |
| [metric]      | 定义跳数，这个一般用不到。当到同一目的地有多条路径的时候，系统会选择metric值最小的路由。 |
| [if]          | 定义网卡。                                                   |
| [interface]   | 网卡的接口号码，在使用route print命令的时候，可以看到该号码。 |

## 2. 常用命令总结

```shell
# 例子1: 要显示IP路由表的完整内容，执行以下命令：
route print

# 例子2: 要显示IP路由表中以10.开始的路由，执行以下命令：
route print 10.*

# 例子3: 要添加默认网关地址为192.168.12.1的默认路由，执行以下命令：
route add 0.0.0.0 mask 0.0.0.0 192.168.12.1

# 例子4: 要添加目标为10.41.0.0，子网掩码为255.255.0.0，下一个跃点地址为10.27.0.1的路由，执行以下命令：
route add 10.41.0.0 mask 255.255.0.0 10.27.0.1

# 例子5: 要添加目标为10.41.0.0，子网掩码为255.255.0.0，下一个跃点地址为10.27.0.1的永久路由，执行以下命令：
route -p add 10.41.0.0 mask 255.255.0.0 10.27.0.1

# 例子6: 要添加目标为10.41.0.0，子网掩码为255.255.0.0，下一个跃点地址为10.27.0.1，跃点数为7的路由，执行以下命令：
route add 10.41.0.0 mask 255.255.0.0 10.27.0.1 metric 7

# 例子7: 要添加目标为10.41.0.0，子网掩码为255.255.0.0，下一个跃点地址为10.27.0.1，接口索引为0x3的路由，执行以下命令：
route add 10.41.0.0 mask 255.255.0.0 10.27.0.1 if 0x3

# 例子8: 要删除目标为10.41.0.0，子网掩码为255.255.0.0的路由，执行以下命令：
route delete 10.41.0.0 mask 255.255.0.0

# 例子9: 要删除IP路由表中以10.开始的所有路由，执行以下命令：
route delete 10.*

# 例子10: 要将目标为10.41.0.0，子网掩码为255.255.0.0的路由的下一个跃点地址由10.27.0.1更改为10.27.0.25，执行以下命令：
route change 10.41.0.0 mask 255.255.0.0 10.27.0.25
```

