---
title: ssh跳板机与代理
top: false
cover: false
toc: true
mathjax: true
categories:
  - ssh
tags:
  - ssh
date: 2022-11-11 10:30:45
password:
summary:
---

# 一、ssh跳板机

## ssh -J 命令

```bash
ssh -J root@172.29.1.236 root@10.226.99.149
# 多主机之间用逗号隔开，如果某个主机的sshd服务不是默认22端口，需要在IP后加“:port”（冒号和端口号）指定主机端口
#mam手册的解释：
-J [user@]host[:port]
             Connect to the target host by first making a ssh connection to the jump host and then establishing a TCP forwarding to the ultimate destination from there.  Multiple jump hops may be
             specified separated by comma characters.  This is a shortcut to specify a ProxyJump configuration directive.
```

## ssh免密码和ssh-copy-id命令

```bash
# 把本机的公钥追到jifeng02的 .ssh/authorized_keys 里
ssh-copy-id -i ~/.ssh/id_rsa.pub jifeng@jifeng02
# windows等效命令
cat ~/.ssh/id_rsa.pub | ssh username@next_jumper 'cat - >> ~/.ssh/authorized_keys'
```

# 二、利用ssh穿越多个跳板机

**1. 在每个节点产生公钥私钥对**

- ssh-keygen -t rsa 运行此命令产生公钥私钥，一路回车可以不设置保护密码
- 检查是否产生了这俩文件 ~/.ssh/id_rsa，~/.ssh/id_rsa.pub
- 记住这个公钥私钥对所属的用户名

**2. 把本机公钥放到每一个直连下一跳的~/.ssh/authorized_keys**

- cat ~/.ssh/id_rsa.pub | ssh username@next_jumper 'cat - >> ~/.ssh/authorized_keys'， 此处需要username的密码
- 尝试ssh username@next_jumper 这时不需要密码应该能登录
- 对中间跳板机之间的每个登录关系做如上验证

**3. 作为访问起点的client需要额外设置**

- 确保ssh client支持ProxyJump语句(openssh 7.5版本之后，mac mojave和ubuntu 18.04开始)
- 创建~/.ssh/config文件，描述跳板机的树形级联拓扑结构(下面举例描述两条路径client->jumper1->jumper2a->jumper3->node1, client->jumper1->jumper2b->jumper4->node2|192.168.103.0/24, 注意，第一级jumper不带ProxyJump语句, Port为22也可以缺省不写)：

```bash
Host jumper1
  User <user_of_jumper1>
  Hostname <ip_of_jumper1>

Host jumper2a
  User <user_of_jumper2a>
  Port 2222
  Hostname <ip_of_jumper2a>
  ProxyJump jumper1

Host jumper2b
  User <user_of_jumper2b>
  Hostname <ip_of_jumper2b>
  ProxyJump jumper1

Host jumper3
  User <user_of_jumper3>
  Hostname <ip_of_jumper3>
  ProxyJump jumper2a

Host jumper4
  User <user_of_jumper4>
  Hostname <ip_of_jumper4>
  ProxyJump jumper2b

Host node1 n1
  User <user_of_node1>
  Hostname <ip_of_node1>
  ProxyJump jumper3

Host node2 n2
  User <user_of_node2>
  Hostname <ip_of_node2>
  ProxyJump jumper4

Host 192.168.103.*
  User <username_on_target>
  ProxyJump jumper4
```

* 参照步骤2把client公钥放置到每个叶子节点<node1, node2, 192.168.103.*>

**5. client设置ControlMaster高速复用ssh连接**

将如下加入到~/.ssh/config文件顶部，第一次ssh登录之后再次ssh登陆，几乎没有无延时。一段时间内重用连接不需要认证过程（对于密码登陆方式就是不要求输入密码了）

```
Host *
  ServerAliveInterval 10
  TCPKeepAlive yes
  ControlPersist yes
  ControlMaster auto # windows不可用
  ControlPath ~/.ssh/master_%r_%h_%p # windows不可用
```

**6.增量同步远程机器的指定目录**

lftp这个老古董可以利用sftp经由ssh的config跳板通道，进行文件拷贝同步等动作。

```bash
#把client本地/root/project/下的文件增量同步到远程n1上的/root/project/
lftp sftp://n1 -e "mirror -R /root/project/"

#把远端n1上的/root/project/下的文件增量同步到client本地/root/project/
lftp sftp://n1 -e "mirror /root/project/"
# lftp还有多线程参数加速同步或者下载，读者可自行发掘。
```

# 三、技巧

### vim 访问远程文件

```bash
vim scp://root@example.com//home/centos/docker-compose.yml
vim scp://example//home/centos/docker-compose.yml
```

# 四、文件拷贝

#五、SSH代理

## 5.1 动态代理

> 动态代理一般用于代理服务器，应用场景为：本地不能直接访问某些地址/端口，但远程机可以。本机通过建立一个指定本机端口，远程机端口不指定（动态）的连接，让本地可以通过该连接去访问那些地址（基于socks4和socks5协议）。

```bash
ssh -o ServerAliveInterval=20 -g -Nf -D 6060 proxy@47.44.161.114               #动态代理
```

| 参数                          | 说明                                                         |
| :---------------------------- | :----------------------------------------------------------- |
| **-o ServerAliveInterval=20** | 代表心跳包，ssh在一段时间没数据后会把连接给断开。            |
| **-g**                        | 允许**其他主机**连接到本机端口进行转发。 如果无效，要设置**本机\**sshd_config文件:\**gatewayports yes** |
| **-N**                        | 不执行远程命令。 仅做端口转发（仅适用于协议版本2）。         |
| **-f**                        | 将ssh切换到后台                                              |
| **-D 6060**                   | 指定以本机哪个端口做为转发端口                               |
| **proxy@47.44.161.114**       | 以指定帐号连接远程机                                         |

## 5.2 反向代理

> 反向代理一般用于内网穿透，应用场景为：本机在防火墙内，并且防火墙未向外开放本机（或本地网络内其他主机）端口，远程机有向外开放的可用端口，本机通过建立一个指定本机（或本地网络内其他主机）端口和远程机端口的连接（也可以理解成端口映射），让外部应用可以通过远程机端口访问本机（或本地网络内其他主机）端口。

```bash
ssh -o ServerAliveInterval=20 -g -Nf -R 5001:localhost:6060 proxy@47.44.161.114 #反向代理
```

| 参数                          | 说明                                                         |
| :---------------------------- | :----------------------------------------------------------- |
| **-o ServerAliveInterval=20** | 代表心跳包，ssh在一段时间没数据后会把连接给断开。            |
| **-g**                        | 允许其**远程机群**连接到**远程机**端口进行转发。 如果无效，要修改**远程机\**sshd_config文件:\**gatewayports yes** |
| **-N**                        | 不执行远程命令。 仅做端口转发（仅适用于协议版本2）。         |
| **-f**                        | 将ssh切换到后台                                              |
| **-R 15001:localhost:6060**   | 反向转发，用值用**:**分隔为三项,格式为： <远程机端口>:<本地机群>:<端口>。 |
| **proxy@47.44.161.114**       | 以指定帐号连接远程机                                         |

## 5.3 **本地代理**

```bash
ssh -o ServerAliveInterval=20 -g -Nf  -L 5001:localhost:6060 proxy@47.44.161.114
```

| 参数                          | 说明                                                         |
| :---------------------------- | :----------------------------------------------------------- |
| **-o ServerAliveInterval=20** | 代表心跳包，ssh在一段时间没数据后会把连接给断开。            |
| **-g**                        | 允许**本地机群**连接到**本机**端口进行转发。 如果无效，要修改**本机\**sshd_config文件:\**gatewayports yes** |
| **-N**                        | 不执行远程命令。 仅做端口转发（仅适用于协议版本2）。         |
| **-f**                        | 将ssh切换到后台                                              |
| **-L 15001:localhost:6060**   | 本地转发，用值用**:**分隔为三项,格式为： <本机端口>:<远程机群>:<端口>。 示例里第二项localhost,代表的是**远程机**，用这种写法一般代表远程主机也只能本机访问该端口 |
| **proxy@47.44.161.114**       | 以指定帐号连接远程机                                         |

# 参考文档

[ssh技巧之跳板机]: https://mp.weixin.qq.com/s/IwLoUB99RTIBnnLm3FWIlw
[ssh隧道代理]: http://wlwang41.github.io/content/ops/ssh%E9%9A%A7%E9%81%93%E4%BB%A3%E7%90%86.html
[SSH Config 那些你所知道和不知道的事]: SSHConfig那些你所知道和不知道的事
[ssh端口转发笔记：ssh反向代理(隧道)、动态代理、本地代理]: https://blog.csdn.net/weixin_30337157/article/details/97996649
[SSH动态代理访问内网]: https://blog.csdn.net/anjingshen/article/details/89055509



