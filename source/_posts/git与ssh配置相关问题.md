---
title: git与ssh配置相关问题
toc: true
mathjax: true
categories:
  - Git
  - 配置 
tags:
  - Git
date: 2022-05-27 21:24:32
password:
summary:
---

# Git SSH相关问题

## 一 ssh生成

### 1、进行配置

```bash
# 检查下自己之前有没有已经生成
ls -al ~/.ssh
#设置Git的user name和email：
git config --global user.name "你的名字"
git config --global user.email "你的邮箱@gmail.com"
```

###  2、生成秘钥

代码参数含义：

* -t 指定密钥类型，默认是 rsa ，可以省略。
* -C 设置注释文字，比如邮箱。
* -f 指定密钥文件存储文件名。

```css
# 生成秘钥
ssh-keygen -t rsa -C "你的邮箱@gmail.com"

# 接着按3个回车
[root@localhost ~]# ssh-keygen -t rsa       <== 建立密钥对，-t代表类型，有RSA和DSA两种
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):   <==密钥文件默认存放位置，按Enter即可
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase):     <== 输入密钥锁码，或直接按 Enter 留空
Enter same passphrase again:     <== 再输入一遍密钥锁码
Your identification has been saved in /root/.ssh/id_rsa.    <== 生成的私钥
Your public key has been saved in /root/.ssh/id_rsa.pub.    <== 生成的公钥
The key fingerprint is:
SHA256:K1qy928tkk1FUuzQtlZK+poeS67vIgPvHw9lQ+KNuZ4 root@localhost.localdomain
The key's randomart image is:
+---[RSA 2048]----+
|           +.    |
|          o * .  |
|        . .O +   |
|       . *. *    |
|        S =+     |
|    .    =...    |
|    .oo =+o+     |
|     ==o+B*o.    |
|    oo.=EXO.     |
+----[SHA256]-----+
```

最后在.ssh目录下(C盘用户文件夹下)得到了两个文件：`id_rsa`（私有秘钥）和`id_rsa.pub（`公有密钥）

### 3、拷贝到剪切板

```bash
#  拷贝到剪切板
cat id_rsa.pub| pbcopy
```

### 4、 测试

```bash
ssh -T git@github.com
```

**以下命令代表成功**

```css
~/blog/source/_posts source*
❯ ssh -T git@github.com
Hi root! You've successfully authenticated, but GitHub does not provide shell access.
```

## 二 多个SSH Key及对应Git配置的详细说明

### 1、应用场景

> 如果有多个[github](https://so.csdn.net/so/search?q=github&spm=1001.2101.3001.7020)账号，不同账号使用不同SSH Key，或者使用多个托管网站（gitee、github、gitlab…）等，不同托管网站使用不同SSH Key，那么就需要生成不同的公钥，然后分别配置，具体操作步骤如下。

### 2、查看已有SSH Key

| 查看已有SSH Key                                              |
| ------------------------------------------------------------ |
| 输入： `ls ~/.ssh/` 能看到你已有的SSH Key                    |
| 输出：`config gitee_id_rsa gitee_id_rsa.pub id_rsa id_rsa.pub known_hosts` |
| 例如上面意思是已有gitee_id_rsa 和 id_rsa 2个key，如果没有则按步骤三生成，如果已有key可进入步骤四。 |
| **注意：已有key即使配置成功也可能使用不了，那么还是生成新的key吧** |

### 3、生成SSH Key

|                                                              |
| ------------------------------------------------------------ |
| 输入： `ssh-keygen -t rsa -C 'xxxx@xx.com' -f ~/.ssh/gitee_id_rsa` |
| xxxx@xx.com 为你的邮箱。 `-f ~/.ssh/gitee_id_rsa` 是指定key的命名为gitee_id_rsa，不指定的话，默认为id_rsa |
| 然后按提示会要求输入密码，密码可以为空，直接回车即可，3次回车后会生成key<br/>**注意，其中画红线的部分记录着key的存储路径，记下来，后面用到：** |
| Your identification has been saved in ~.ssh/gitee_id_rsa.    |

### 4、Git配置多个SSH Key

#### 4.1 配置文件地址

1. 用户配置文件 (~/.ssh/config)
2. 系统配置文件 (/etc/ssh/ssh_config)

#### 4.2 配置

打开目录 `.ssh` ，在该目录下，新建一个`config`文件，注意：该文件名称为config，没有后缀 用记事本打开，输入：

```bash
# gitee
Host gitee.com
HostName gitee.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/gitee_id_rsa

# github_A
Host github.com
HostName github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/github_A_id_rsa

# github_B
Host github.com
HostName github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/github_B_id_rsa
```

上文意思是，gitee.com 可使用gitee_id_rsa公钥，github可使用github_A_id_rsa 和 github_B_id_rsa 公钥。

#### 4.3 添加认证到ssh

```bash
ssh-add ~/.ssh/github_B_id_rsa
```



## 三 ssh config 参数介绍

### 1、 Host

`Host`配置项标识了一个配置区段。

`ssh`配置项参数值可以使用通配符：`*`代表0～n个非空白字符，`?`代表一个非空白字符，`!`表示例外通配。

我们可以在系统配置文件中看到一个匹配所有`host`的默认配置区段：

```
$ cat /etc/ssh/ssh_config | grep '^Host'
Host *
```

这里有一些默认配置项，我们可以在用户配置文件中覆盖这些默认配置。

### 2、 GlobalKnownHostsFile

指定一个或多个全局认证主机缓存文件，用来缓存通过认证的远程主机的密钥，多个文件用空格分隔。默认缓存文件为：/etc/ssh/ssh_known_hosts, /etc/ssh/ssh_known_hosts2.

### 3、 HostName

指定远程主机名，可以直接使用数字IP地址。如果主机名中包含 ‘%h’ ，则实际使用时会被命令行中的主机名替换。

### 4、 IdentityFile

指定密钥认证使用的私钥文件路径。默认为 ~/.ssh/id_dsa, ~/.ssh/id_ecdsa, ~/.ssh/id_ed25519 或 ~/.ssh/id_rsa 中的一个。文件名称可以使用以下转义符：

```bash
'%d' 本地用户目录
'%u' 本地用户名称
'%l' 本地主机名
'%h' 远程主机名
'%r' 远程用户名
```

可以指定多个密钥文件，在连接的过程中会依次尝试这些密钥文件

### 5、 Port

指定远程主机端口号，默认为 22 。

### 6、 User

指定登录用户名。

### 7、 UserKnownHostsFile

指定一个或多个用户认证主机缓存文件，用来缓存通过认证的远程主机的密钥，多个文件用空格分隔。默认缓存文件为： ~/.ssh/known_hosts, ~/.ssh/known_hosts2.

还有更多参数的介绍，可以参看用户手册：

```bash
man ssh config
```

### 8、示例

> - 以下连接为例：

```bash
Git 服务器： ssh.test.com
端口号： 19003
密钥文件： ~/.ssh/id_rsa_test

Git 服务器：192.168.173.1 
端口号： 19004
密钥文件： ~/.ssh/id_rsa_gitlab
```

- 有如下配置文件：

```bash
$ vim ~/.ssh/config
Host sshtest
    HostName ssh.test.com
    Port 19003
    IdentityFile ~/.ssh/id_rsa_test

Host 192.168.173.1 
    HostName 192.168.173.1 
    Port 19004
    IdentityFile ~/.ssh/id_rsa_gitlab

```

## 四、一个小坑

### 4.1 Gogs 服务器ssh连接授权失败问题

**报错信息**

> 不断提示权限问题，让输入密码

```css
git clone git@gitlab.xxx.com:19003:USERNAME/project-name.git
Cloning into 'project-name'...
git@gitlab.xxx.com's password:
Permission denied, please try again.
git@gitlab.xxx.com's password:
```

**解决办法**

> 添加config文件见上，添加如下配置，并且URl里面19003端口号是不需要写的

```css
## 示例配置
# 域名地址的别名
Host 39.106.18.30
# 这个是真实的域名地址            
Hostname 39.106.18.30
# 这里是id_rsa的目录位置                
IdentityFile ~/.ssh/id_rsa
# 默认是22，如果是其他端口，一定要配置
Port 19003
```

### 4.2 HTTP一直输入账号密码问题

提示输入账号和密码的，为账号认证失败，使用`git config --global credential.helper store`命令，可输入一次密码后永久存储

**原因**

可能执行了`git config --global credential.helper manager` 命令

通过`git config --list` 查看

**解决**

使用使用 `git config --system --unset credential.helper`

或者  `git config --global --unset credential.helper`

执行之后继续检查 `git config --list` 查看是否存在 manager 

如果还存在可能设置了全局 那么执行`git config --global --unset credential.helper`。总之，只要是还存在的话，就要想办法将它去掉。

>  `git config --global credential.helper store` （这个指令执行后，会要求第一次输入密码，然后账号和密码会被缓存到.git-credentials文件中，后续就不用再输入账号密码了）
>
> 
>
> `cat ~/.git-credentials`

查看下你的用户目录下有个.git-credentials文件，同时存了你的账号和密码。查看发现确实生成了相应的文件，存了账号和密码

```bash
$ cat ~/.git-credentials
https://test%40qq.com:123123%2a@github.com
```

如果出现下面这种错误：你可以把~/.git-credentials这个文件删除掉，重新输入一次密码就行了。

```bash
git push origin master
remote: Invalid username or password.
fatal: Authentication failed for 'https://github.com/shamogulang/git-learn.git/'
```

