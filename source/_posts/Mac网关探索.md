---
title: Mac网关探索
top: false
cover: false
toc: true
mathjax: true
categories:
  - Mac
tags:
  - Mac
date: 2021-11-22 17:59:42
password:
summary:
---

# Mac网关路由

> 目的为作为网关方便其他计算机通过wifi访问内网

1. 打开MAC笔记本的路由转发功能

```bash
#开启路由转发
sysctl net.inet.ip.forwarding=1 #立刻生效
sudo echo 'net.inet.ip.forwarding=1' >> /etc/sysctl.conf #永久有效
```

2. 查看本地外网网卡与内网网卡

<!--vpn创建虚拟的网卡设备，名称通常是以utun为前缀-->

执行命令ifconfig找到两段内容

我的内网网卡位ens0(192.168.8.108)，外网网卡位ens6(172.16.7.193)

3. 修改pfctl的配置文件/etc/pf.conf  ==最好先进行备份==

```ruby
# 编写NAT规则:
# 这条规则是从本地网络en0到any数据包的源地址修改为en6是的地址
nat on en6 from en0:network to any -> (en6)
```

4. 继续，把上面写好的NAT规则写入/etc/pf.conf

备注：规则插入的位置是有限制的，NAT规则必须在nat-anchor "com.apple/*"后面

5.  重新加载/etc/pf.conf

```bash
sudo pfctl -e -f /etc/pf.conf
```

6. pfctl -sn命令检查是否成功

# 参考

[pfctl 简介](https://www.freebsd.org/cgi/man.cgi?query=pfctl(8))

[pf.conf规则说明文档](https://murusfirewall.com/Docum)

 [pf.conf规则说明中文文档](http://blog.sina.com.cn/s/blog_56f75c1b0102uwa9.html)

[如何将Mac（OS X Yosemite）设置为互联网网关](https://mlog.club/article/5273641)

[Mac OS X 設定NAT與port forwarding](https://jerry.thesolarsystems.net/?p=860)

[如何使用Linux作为网关？](https://qastack.cn/unix/222054/how-can-i-use-linux-as-a-gateway)

[Linux 服务器做网关](https://blog.csdn.net/jackliu16/article/details/80155558)

