---
title: windows Java多版本切换
top: false
cover: false
toc: true
mathjax: true
categories:
  - windows
tags:
  - java
  - windows 
date: 2024-04-22 10:37:48
password:
summary:
---

# windows Java多版本切换

目前存在以下解决方案

1. [**vfox**](https://github.com/version-fox/vfox) ***推荐***
2. scoop
3. [sdkman](https://sdkman.io/) windows 支持不友好 卡顿，很慢
4. [Jabba](https://github.com/shyiko/jabba)  没研究
5. [jenv](https://github.com/jenv/jenv) windows 支持不友好

总结：

1. vfox作用在用户变量，每次修改环境变量的值
2. scoop 可以作用在系统变量，但是每个版本都会生成一个，目前体验起来不好用
3. sdkman 可以自己设置环境变量后使用这个切换，缺点是太卡了，除了卡就是多一步手动切换的过程

## 1 vfox

> 原理是增加用户变量，每次切换更新这个用户变量
>
> 似乎不能修改系统变量

[vfox Java 插件 文档](https://github.com/version-fox/vfox-java/blob/main/README_CN.md)

[vfox使用文档](https://vfox.lhan.me/)

## 2 scoop

> 切换起来比较笨，也是在环境变量中增加配置，但是他加多余切有问题
>
> sudo 命令能够添加系统变量

### 什么是 Scoop ？

Scoop 是 Windows 的命令行安装程序，是一个强大的包管理工具。可以在 github 上找到其项目的相关信息，[项目地址](https://www.mobaijun.com/go.html?u=aHR0cHM6Ly9naXRodWIuY29tL1Njb29wSW5zdGFsbGVyL1Njb29w),Scoop 等一系列包管理器的诞生，第一大便利就是省去了繁琐的「搜索 - 下载 - 安装」的步骤，让我们能够通过「一行代码」急速安装。💪

同时，用 Scoop 来安装和管理我们的软件：

- 集搜索、下载、安装、更新软件于一体：极大的降低了安装维护一个软件的成本，我们甚至不必在软件本身的复杂菜单中寻找那个更新按钮来更新软件自己
- 将软件干干净净的安装到电脑的「用户文件夹」下：这样既不会污染路径也不会请求不必要的权限（UAC）
- 在卸载软件的时候，能够尽量清空软件在电脑上存储的任何数据和痕迹

Scoop 最适合安装那种干净、小巧、开源的软件。并且，Scoop 也极度适合为开发者配置开发环境，不过这些很多都涉及到进阶使用技巧。下面先从基础的安装方法开始介绍。

### Scoop 的安装配置

安装 Scoop 很简单，不过要先确定一些基础环境是否符合安装要求：

- Windows 版本不低于 Windows 7
- Windows 中的 PowerShell 版本不低于 PowerShell 3
- 你能 **正常、快速** 的访问 GitHub 并下载上面的资源，GitHub 访问加速可以参考[【工具系列】FastGithub–GitHub 加速工具 | 框架师](https://www.mobaijun.com/go.html?u=aHR0cHM6Ly93d3cubW9iYWlqdW4uY29tL3Bvc3RzLzEwOTUzMDk4MDIuaHRtbA==)
- 你的 Windows 用户名为英文（Windows 用户环境变量中路径值不支持中文字符）

然后右键开始菜单按钮，在右键菜单中打开 PowerShell：

![img](0.png)

在 PowerShell 中输入下面内容，保证允许本地脚本的执行：

```bash
$ set-executionpolicy remotesigned -scope currentuser
```

然后执行下面的命令安装 Scoop：

```bash
$ Invoke-Expression (New-Object System.Net.WebClient).DownloadString('https://get.scoop.sh')
# 或者使用下面这条命令
$ iwr -useb get.scoop.sh | iex
```

> 这样有个问题既是安装在默认路径，在 `C:\ProgramData\scoop` 目录下，如何自定义安装位置呢？

```bash
$env:SCOOP='D:\APP\Scoop'
---------------------------
[Environment]::SetEnvironmentVariable('SCOOP', $env:SCOOP, 'User')
```

> 上面两条指令分别输入到 PowerShell 即可，第一条指令表示配置安装 Scoop 的目标路径，第二条指令写入配置，然后在执行上面的安装命令就可以了。

配置环境变量，将 `D:\APP\Scoop\shims` 目录添加到系统 `Path` 目录下，全局调用 Scoop 指令。

```bash
$ scoop help
Usage: scoop <command> [<args>]

Some useful commands are:

alias       Manage scoop aliases
bucket      Manage Scoop buckets
cache       Show or clear the download cache
cat         Show content of specified manifest.
checkup     Check for potential problems
cleanup     Cleanup apps by removing old versions
config      Get or set configuration values
create      Create a custom app manifest
depends     List dependencies for an app
export      Exports (an importable) list of installed apps
help        Show help for a command
hold        Hold an app to disable updates
home        Opens the app homepage
info        Display information about an app
install     Install apps
list        List installed apps
prefix      Returns the path to the specified app
reset       Reset an app to resolve conflicts
search      Search available apps
status      Show status and check for new app versions
unhold      Unhold an app to enable updates
uninstall   Uninstall an app
update      Update apps, or Scoop itself
virustotal  Look for app's hash on virustotal.com
which       Locate a shim/executable (similar to 'which' on Linux)
```

这样就表明 Scoop 已经成功安装了。`scoop help` 这个命令就是 Scoop 的使用说明书，如果我们记不住某个命令怎么执行，也可以通过 `scoop help` 来唤起这个命令参考说明。

### Scoop 使用方法

Scoop 默认安装的主存储桶可能会不够用，所以需要添加一个额外的存储桶，这里的存储桶就相当于是软件商店概念，其他存储桶列表可以参考这个链接[ 传送门](https://rasa.github.io/scoop-directory/)，也可以通过如下命令进行搜索；

```bash
$ scoop bucket known
main
extras
versions
nirsoft
php
nerd-fonts
nonportable
java
games
```

以上官方认可的存储桶可以通过下面这个命令直接添加：

```bash
$ scoop bucket add <bucketname>
```

顺便强推 `extras` 这个存储桶，包含了大多数我们平常使用到的软件。

```bash
顺便强推 extras 这个存储桶，包含了大多数我们平常使用到的软件。
```

### Scoop 基础语法

最基础的使用方法很简单，这里就不做赘述了，直接上命令列表：

```bash
# 搜索软件 
$ scoop search <app>

# 安装软件
$ scoop install <app>

# 查看软件详细信息
$ scoop info <app>

# 查看已安装软件
$ scoop list

# 卸载软件，-p删除配置文件。
$ scoop uninstall <app>

# 更新 scoop 本体和软件列表
$ scoop update

# 更新所有已安装的软件
$ scoop update *

# 检查 scoop 的问题并给出解决问题的建议
$ scoop checkup

# 查看命令列表
$ scoop help

# 查看命令帮助说明
$ scoop help <command>
```

### 清理安装包缓存

Scoop 会保留下载的安装包，对于卸载后又想再安装的情况，不需要重复下载。但长期累积会占用大量的磁盘空间，如果用不到就成了垃圾。这时可以使用 `scoop cache` 命令来清理。

```bash
# 显示安装包缓存
$ scoop cache show

# 删除指定应用的安装包缓存
$ scoop cache rm <app>

# 删除所有的安装包缓存
$ scoop cache rm *

# 如果你不希望安装和更新软件时保留安装包缓存，可以加上 -k 或 --no-cache 选项来禁用缓存：
$ scoop install -k <app>
------------------------
$ scoop update -k *
```

### 删除旧版本软件

当软件被更新后 Scoop 还会保留软件的旧版本，更新软件后可以通过 `scoop cleanup` 命令进行删除。

- 删除指定软件的旧版本

```bash
$ scoop cleanup <app>
```

### 全局安装

全局安装就是给系统中的所有用户都安装，且环境变量是系统变量，对于需要设置系统变量的一些软件就需要全局安装，比如 Node.js、Python ，否则某些情况会出现无法找到命令的问题。

使用 `scoop install <app>` 命令加上 `-g` 或 `--global` 选项可对软件进行全局安装，全局安装需要管理员权限，所以需要提前以管理员权限运行的 Pow­er­Shell 。更简单的方式是先安装 `sudo`，然后用 `sudo` 命令来提权执行：

```bash
$ scoop install sudo
$ sudo scoop install -g <app>
```

> 达成在 Win­dows 上使用 `sudo` 的成就

使用 `scoop list` 命令查看已装软件时，全局安装的软件末尾会有 `*global*` 标志。

```bash
➜ scoop list
Installed apps:
  git 2.26.2.windows.1 *global*
  nodejs-lts 12.17.0 *global*
```

此外对于全局软件的更新和卸载等其它操作，都需要加上 `-g` 选项：

- 更新所有软件（且包含全局软件）

```bash
$ sudo scoop update -g *
```

- 卸载全局软件

```bash
$ sudo scoop uninstall -g <app>
```

- 卸载全局软件（并删除配置文件）

```bash
$ sudo scoop uninstall -gp <app>
```

- 删除所有全局软件的旧版本

```bash
$ sudo scoop cleanup -g *
```

- 删除所有全局软件的旧版本（并清除安装包包缓存）

```bash
$ sudo scoop cleanup -gk *
```

### 代理设置

Scoop 默认使用的是系统代理，如果你想手动指定代理，可以输入下面的命令。需要注意的是只支持 `http` 协议。

```bash
$ scoop config proxy 127.0.0.1:20080
```

> 设置完可以通过 `scoop config proxy` 查看。

如果你想取消代理，那么输入下面的命令，这将会恢复使用系统代理。

```bash
$ scoop config rm proxy
```

### 开启多线程下载

使用 Scoop 安装 Aria2 后，Scoop 会自动调用 Aria2 进行多线程加速下载。

```bash
$ scoop install aria2
```

使用 `scoop config` 命令可以对 Aria2 进行设置，比如 `scoop config aria2-enabled false` 可以禁止调用 Aria2 下载。以下是与 Aria2 有关的设置选项：

- `aria2-enabled`: 开启 Aria2 下载，默认`true`
- `aria2-retry-wait`: 重试等待秒数，默认`2`
- `aria2-split`: 单任务最大连接数，默认`5`
- `aria2-max-connection-per-server`: 单服务器最大连接数，默认`5` ，最大`16`
- `aria2-min-split-size`: 最小文件分片大小，默认`5M`

在这里推荐以下优化设置，单任务最大连接数设置为 `32`，单服务器最大连接数设置为 `16`，最小文件分片大小设置为 `1M`

```bash
$ scoop config aria2-split 32
$ scoop config aria2-max-connection-per-server 16
$ scoop config aria2-min-split-size 1M
```

### 常用命令总结

看到这里可能有很多小伙伴已经懵逼了，最后总结一波 Scoop 在日常使用中的常用命令：

```bash
# 更新 scoop 及软件包列表
$ scoop update
# 非全局安装（并禁止安装包缓存）
$ scoop install -k <app>
# 全局安装（并禁止安装包缓存）
$ sudo scoop install -gk <app>
# 卸载非全局软件（并删除配置文件）
$ scoop uninstall -p <app>
# 卸载全局软件（并删除配置文件）
$ sudo scoop uninstall -gp <app>
# 更新所有非全局软件（并禁止安装包缓存）
$ scoop update -k *
# 更新所有软件（并禁止安装包缓存）
$ sudo scoop update -gk *
# 删除所有旧版本非全局软件（并删除软件包缓存）
$ scoop cleanup -k *
# 删除所有旧版本软件（并删除软件包缓存）
$ sudo scoop cleanup -gk *
# 清除软件包缓存
$ scoop cache rm *
```

### 参考资料

[「一行代码」搞定软件安装卸载，用 Scoop 管理你的 Windows 软件 - 少数派](https://sspai.com/post/52496)

[scoop教程](https://www.mobaijun.com/posts/908521329.htm)

## 3 sdkman

### **GitBash安装并配置工具链**

安装SDKMAN前需要安装GitBash，GitBash的安装直接去[官网](https://github.com/git-for-windows/git/releases/download/v2.40.1.windows.1/Git-2.40.1-64-bit.exe)下载一个Windows版本的安装包，安装好就可以了。这里就不演示了。

安装好GitBash之后，需要补充下工具链，默认情况下GitBash缺少了zip命令，需要自行安装下，可以从[这里](https://sourceforge.net/projects/gnuwin32/files/zip/3.0/zip-3.0-bin.zip/download)下载安装包，然后将压缩包里面的`bin\zip.exe`解压到Git安装目录下的`mingw\bin`中。例如我的Git安装`D:\Programs\Git`目录，就需要将zip.exe解压到`D:\Programs\Git\mingw\bin.exe`。

安装

```bash
curl -s "https://get.sdkman.io" | bash
```

配置Java环境变量

C:\Users\25337\.sdkman\candidates\java\current

切换的原理就是改变current 的软连接位置

### 参考资料

https://www.jianshu.com/p/ddce36a50720

https://blog.uusite.com/system/software/423.html

## 4 Jabba

## 5 Jenv

https://www.cnblogs.com/tysec/p/16055754.html
