---
title: win10 安装 wsl2
top: false
cover: false
toc: true
mathjax: true
categories:
  - WSL
tags:
  - Win10 安装WSL
date: 2021-04-12 23:05:52
password:
summary:
---

## 注意事项

Windows 10 的 WSL 2 需要依赖于， Microsoft Store中的应用。适用于 Linux 的 Windows 子系统只能在系统驱动器（通常是 C: 驱动器）中运行，所以注意C盘的空间。

# 一、在 Windows 10 上安装 Hyper-V

Docker Desktop 想要在Windows上运行，需要依赖于Windows的Hyper-V模块。所以首先就要启用Hyper-V。

启用 Hyper-V 以在 Windows 10 上创建虚拟机。可以通过多种方式启用 Hyper-V，包括使用 Windows 10 控制面板、PowerShell 或使用部署映像服务和管理工具 (DISM)。

## (1) 检查要求

- Windows 10 企业版、专业版或教育版
- 具有二级地址转换 (SLAT) 的 64 位处理器。
- CPU 支持 VM 监视器模式扩展（Intel CPU 上的 VT-c）。
- 最少 4 GB 内存。

> 请勿在 Windows 10 家庭版上安装 Hyper-V。

## (2) 方式一、使用 PowerShell 启用 Hyper-V

1. 以**管理员身份**打开 PowerShell 控制台。
2. 运行以下命令：

```
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

> 如果无法找到此命令，请确保你以管理员身份运行 PowerShell。
>
> 安装完成后，请重启操作系统。

## (3) 方式二、使用 CMD 和 DISM 启用 Hyper-V

部署映像服务和管理工具 (DISM) 可帮助配置 Windows 和 Windows 映像。在众多应用程序中，DISM 可以在操作系统运行时启用 Windows 功能。

使用 DISM 启用 Hyper-V 角色：

1. 以**管理员身份**打开 PowerShell 或 CMD 会话。
2. 键入下列命令：

```
DISM /Online /Enable-Feature /All /FeatureName:Microsoft-Hyper-V
```

![命令](image-20210424112327542.png)

## (4) 方式三、通过“设置”启用 Hyper-V

- 右键单击 Windows 按钮并选择“应用和功能”。（左下角Windows图标）
- 选择相关设置下右侧的“程序和功能”。
- 选择“打开或关闭 Windows 功能”。
- 选择“Hyper-V”，然后单击“确定”。

![设置启用Hyper-V](image-20210424112200782.png)

> 安装完成后，系统会提示你重启计算机。

# 二、适用于 Linux 的 Windows 子系统安装

## (1) 安装适用于 Linux 的 Windows 子系统

必须先启用“适用于 Linux 的 Windows 子系统”可选功能，然后才能在 Windows 上安装 Linux 分发版。

以**管理员身份**打开 PowerShell 并运行：

```
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

> 若要仅安装 WSL 1，现在应重启计算机并继续安装所选的 Linux 分发版，否则请等待重启并继续更新到 WSL 2

## (2) 更新到 WSL 2

### 2.1 若要更新到 WSL 2，必须满足以下条件：

- 运行 Windows 10（已更新到版本 2004 的内部版本 19041 或更高版本）。
- 通过按 Windows 徽标键 + R， 检查你的 Windows 版本，然后键入 winver，选择“确定” 。 （或者在 Windows 命令提示符下输入 ver 命令）。 如果内部版本低于 19041，请更新到最新的 Windows 版本。

### 2.2 启用“虚拟机平台”可选组件

安装 WSL 2 之前，必须启用“虚拟机平台”可选功能。

以**管理员身份**打开 PowerShell 并运行：

```
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

> 重新启动计算机，以完成 WSL 安装并更新到 WSL 2。

### 2.3 将 WSL 2 设置为默认版本

安装新的 Linux 分发版时，请在 Powershell 中运行以下命令，以将 WSL 2 设置为默认版本：

```
wsl --set-default-version 2
```

## (3) 安装所选的 Linux 分发版

1. 打开 [Microsoft Store](https://aka.ms/wslstore)，并选择你偏好的 Linux 分发版 (如果上述连接打开有错，请直接打开Microsoft Store搜索)。


单击以下链接会打开每个分发版的 Microsoft Store 页面，(如果下述连接打开有错，请直接打开Microsoft Store搜索)。：

- [Ubuntu 16.04 LTS](https://www.microsoft.com/store/apps/9pjn388hp8c9)
- [Ubuntu 18.04 LTS](https://www.microsoft.com/store/apps/9N9TNGVNDL3Q)
- [openSUSE Leap 15.1](https://www.microsoft.com/store/apps/9NJFZK00FGKV)
- [SUSE Linux Enterprise Server 12 SP5](https://www.microsoft.com/store/apps/9MZ3D1TRP8T1)
- [SUSE Linux Enterprise Server 15 SP1](https://www.microsoft.com/store/apps/9PN498VPMF3Z)
- [Kali Linux](https://www.microsoft.com/store/apps/9PKR34TNCV07)
- [Debian GNU/Linux](https://www.microsoft.com/store/apps/9MSVKQC78PK6)
- [Fedora Remix for WSL](https://www.microsoft.com/store/apps/9n6gdm4k2hnc)
- [Pengwin](https://www.microsoft.com/store/apps/9NV1GV1PXZ6P)
- [Pengwin Enterprise](https://www.microsoft.com/store/apps/9N8LP0X93VCP)
- [Alpine WSL](https://www.microsoft.com/store/apps/9p804crf0395)

1. 在分发版的页面中，选择“获取”。

![image-20210424112428253](image-20210424112428253.png)

## (4) 设置新分发版

首次启动新安装的 Linux分发版时，将打开一个控制台窗口。（就是之前安装的应用）
![image-20210424112449354](image-20210424112449354.png)

系统会要求你等待一分钟或两分钟，以便文件解压缩并存储到电脑上。未来的所有启动时间应不到一秒。

然后，需要为新的 Linux 分发版创建用户帐户和密码。
![image-20210424112515287](image-20210424112515287.png)

## (5) 将分发版版本设置为 WSL 1 或 WSL 2

可以打开 PowerShell 命令行并输入以下命令（仅在 Windows 内部版本 19041 或更高版本中可用），来检查分配给每个已安装的 Linux 分发版的 WSL 版本：

```
wsl -l -v
```

或

```
wsl --list --verbose
```

> 通过以上命令，就可以查看刚刚已经安装的Linux发行版本，以及当前的WSL版本

![查看刚刚已经安装的Linux发行版本](image-20210424112535737.png)

若要将分发版设置为受某一 WSL 版本支持，请运行：

```
wsl --set-version <distribution name> <versionNumber>
```

请确保将 <distribution name> 替换为你的分发版的实际名称，并将 <versionNumber> 替换为数字“1”或“2”。 可以随时更改回 WSL 1，方法是运行与上面相同的命令，但将“2”替换为“1”。

此外，如果要使 WSL 2 成为你的默认体系结构，可以通过此命令执行该操作：

```
wsl --set-default-version 2
```

这会将安装的任何新分发版的版本设置为 WSL 2。

> Docker Desktop 需要的就是 WSL 2

# 三、win10启动ubuntu1804报错 参考的对象类型不支持尝试的操作

**问题描述:** 

> 从terminal启动ubuntu1804报错: 参考的对象类型不支持尝试的操作. 直接启动ubuntu1804也不行

**解决方法:** 

> 以管理员身份打开Windows PowerShell, 然后执行`netsh winsock reset`, 重启电脑即可, 如下图所示

# 四、安装Windows Terminal

- 打开 Microsoft Store，搜索 Terminal，安装 Windows Terminal，用于后面和 WSL 子系统交互。

# 五、安装配置 Linux 发行版

> 由于默认情况下我们不知道 root 用户的密码，所以如果我们想要使用 root 用户的话可以使用 passwd 命令为 root 用户设置一个新的密码，同时为了避免sudo切换root是需要输入密码，把自己配置的用户名加到sudo免密中，命令如下：

```shell
 # 替换leap为自己单独配置的用户名 
 sudo echo "leap ALL=(ALL:ALL) NOPASSWD: ALL" >>/etc/sudoers 
```

默认的安装源相对国内很慢，我们更换源到阿里云，登录到ubuntu到操作如下：

```shell
cp /etc/apt/sources.list /etc/apt/sources.list.bak

echo "deb http://mirrors.aliyun.com/ubuntu/ focal main restricted
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted
deb http://mirrors.aliyun.com/ubuntu/ focal universe
deb http://mirrors.aliyun.com/ubuntu/ focal-updates universe
deb http://mirrors.aliyun.com/ubuntu/ focal multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-updates multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted
deb http://mirrors.aliyun.com/ubuntu/ focal-security universe
deb http://mirrors.aliyun.com/ubuntu/ focal-security multiverse">/etc/apt/sources.list
```

执行更新：

```
apt update && apt upgrade -y
```

# 六、安装docker

> 因为wsl2已经完整使用了linux内核了，此种方式和先前在linux虚拟机安装docker类似，步骤如下：

```shell
 curl -fsSL https://get.docker.com -o get-docker.sh
 sudo sh get-docker.sh
 sudo service docker start
```

> 执行脚本安装过程中，脚本提示“**建议使用Docker Desktop for windows**”，20s内按Ctrl+C会退出安装，所以需要等待20s，另外此种方式需要访问外网。

检查docker安装正常

```shell
# 检查dockerd进程启动
service docker status
ps aux|grep docker
# 检查拉取镜像等正常
docker pull busybox
docker images
```

> **注意**：不同于完全linux虚拟机方式，WLS2下通过`apt install docker-ce`命令安装的docker无法启动，因为WSL2方式的ubuntu里面没有systemd。上述官方get-docker.sh安装的docker，dockerd进程是用ubuntu传统的init方式而非systemd启动的。

```shell
vim /lib/systemd/system/docker.service

### 修改文件
[Service]
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock -H fd:// --containerd=/run/containerd/containerd.sock
### 上面这一行,主要是增加了`-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock`


1. vim /etc/default/docker

2. 修改启动配置文件，如下：

# 开启远程访问 -H tcp://0.0.0.0:2375
# 开启本地套接字访问 -H unix:///var/run/docker.sock
DOCKER_OPTS="-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock"
重启Docker
sudo service docker restart

```



## 配置docker远程tcp



# 七、修改WSL默认登录用户

> 默认的root没有密码

```shell
ubuntu config --default-user root
```

# 八、WSL用户管理

## 1 root用户下的用户管理

```shell
useradd username					#创建用户username
passwd username					    #给已创建的用户username设置密码
usermod --help					    #修改用户这个命令的相关参数
userdel username					#删除用户username
rm -rf username					    #删除用户username所在目录
```

## 2 用户切换

```shell
sudo -s					            #切换到root
su username					        #切换到用户username
exit					            #退出当前用户，返回root用户
```

## 3 用户组

```shell
groupadd usergroup					#组的添加
groupdel usergroup					#组的删除
```

# 九、wsl设置root密码

```shell
sudo  passwd root
```

# 十、WIn10进入WSL文件系统

## 1 在资源管理器中访问

```shell
\\wsl$\ubuntu
```

## 2 wsl 设置访问快捷方式

> 创建软链接

```shell
ln -s /mnt/d/docker ~/win10
```



# 参考文章

* [参考win10安装wsl2 - 1](https://segmentfault.com/a/1190000038911660)
* [参考win10安装wsl2- 2](https://blog.csdn.net/yushuzhen2008/article/details/104944579)

* [参考wsl2安装docker](https://www.cnblogs.com/360linux/p/13662355.html)

* [安装脚本](gist.github.com/xiaopeng163/f3e72bb1990860859076985d5a723cba)
* [wsl网络问题](https://blog.csdn.net/swordsm/article/details/107948497)