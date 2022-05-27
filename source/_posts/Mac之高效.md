---
title: Mac之高效
top: false
cover: false
toc: true
mathjax: true
categories:
  - mac	
tags:
  - Mac
date: 2021-11-12 11:33:39
password:
summary:
---

# 一、快速跳转到指定目录的oh-my-zsh插件：z与wd

> 前提，你必须要先安装iTerm2和oh my zsh

试想一下这种场景：你想要修改nginx的配置，只知道大概在`/usr/local`目录下,不太清楚具体路径

你可能在终端这样输入：

```
cd /usr
ll
cd local
ll
cd etc
ll
cd nginx
ll
vim nginx.conf
```

看完这篇文章的介绍，你的生活将发生翻天覆地的变化。

如果你的电脑安装了`iterm2`和`oh－my－zsh`，再安装`z`或`wd`插件之后
你只需要执行：

```
z nginx
```

或

```
wd nginx
```

就能一秒直达`nginx`目录

# 安装`z`和`wd`插件

安装zsh的插件

1. 打开用户目录下的`.zshrc`文件
2. 修改`.zshrc`文件,在plugins后面的括号中加入`z` `wd`，两个插件的功能是类似的，选择一个安装就可以了

```
plugins=(git z wd)
```

1. 保存并重新加载`.zshrc`文件

# `z`命令

安装完`z`插件后

输入命令 `z` 即可查看你最近访问最频繁的目录

按`z`+目录名称的部分内容即可跳转目录


# `wd`命令

常用命令：

```
wd                  //查看所有可用命令
wd add (label_name) //标记目录
wd rm (label_name)  //去除目录标记
wd list             //查看所有标记
```

使用方法很简单
进入一个你觉得很常用的目录,如上面提到的`usr/local/etc/nginx`,执行`wd add nginx`,相当于告诉终端：我喜欢这个目录，帮我记住它，什么还要取个名字，那就叫`nginx`吧。以后在任何位置，你只需要执行`wd nginx`，就能直接返回这个目录了

# fd命令

## 简单搜索

fd 旨在帮助你轻松找到文件系统中的文件和文件夹。你可以用 fd 带上一个参数执行最简单的搜索，该参数就是你要搜索的任何东西。例如，假设你想要找一个 Markdown 文档，其中包含单词 services 作为文件名的一部分：

```bash
$ fd services
downloads/services.md
```

如果仅带一个参数调用，那么 fd 递归地搜索当前目录以查找与莫的参数匹配的任何文件和/或目录。使用内置的 find 命令的等效搜索如下所示：

```bash
$ find . -name 'services'
downloads/services.md
```

## **文件和文件夹**

您可以使用 -t 参数将搜索范围限制为文件或目录，后面跟着代表你要搜索的内容的字母。例如，要查找当前目录中文件名中包含 services 的所有文件，可以使用：

```ruby
$ fd -tf services
downloads/services.md
```

以及，找到当前目录中文件名中包含 services 的所有目录：

```ruby
$ fd -td services
applications/services
library/services
```

如何在当前文件夹中列出所有带 .md 扩展名的文档？

```ruby
$ fd .md
administration/administration.md
development/elixir/elixir_install.md
readme.md
sidebar.md
linux.md
```

从输出中可以看到，fd 不仅可以找到并列出当前文件夹中的文件，还可以在子文件夹中找到文件。很简单。

你甚至可以使用 -H 参数来搜索隐藏文件：

```ruby
fd -H sessions .
.bash_sessions
```

## **指定目录**

如果你想搜索一个特定的目录，这个目录的名字可以作为第二个参数传给 fd：

```ruby
$ fd passwd /etc
/etc/default/passwd
/etc/pam.d/passwd
/etc/passwd
```

在这个例子中，我们告诉 fd 我们要在 etc 目录中搜索 passwd 这个单词的所有实例。

## **全局搜索**

如果你知道文件名的一部分，但不知道文件夹怎么办？假设你下载了一本关于 Linux 网络管理的书，但你不知道它的保存位置。没有问题：

```
fd Administration /
```

# fzf插件

## 配置

### 核心命令 FZF_DEFAULT_COMMAND

对于使用 fzf 来查找文件的情况，fzf 其实底层是调用的 Unix 系统 find 命令，如果你觉得 find 不好用也可以使用其它查找文件的命令行工具「我使用 fd」。注意：对原始命令添加一些参数应该在这个环境变量里面添加

比如说我们一般都会查找文件 -type f，通常会忽略一些文件夹/目录 --exclude=...，下面是我的变量

```shell
export FZF_DEFAULT_COMMAND="fd --exclude={.git,.idea,.vscode,.sass-cache,node_modules,build,target,logs,dist} --type f"

# 使用 rg 进行搜索
export FZF_DEFAULT_COMMAND='rg --files --hidden'
```

### 界面展示 FZF_DEFAULT_OPTS

界面展示这些参数在 `fzf --help` 中都有，按需配置即可

```bash
export FZF_DEFAULT_OPTS="--height 40% --layout=reverse --preview '(highlight -O ansi {} || cat {}) 2> /dev/null | head -500'"
# 目前使用如下
export FZF_DEFAULT_OPTS="--height 99% -e --layout=reverse --preview '(bat --style=numbers --color=always {} ||  highlight -O ansi -l     {} || coderay {} || rougify {} || cat {}) 2> /dev/null '  --color 'fg:#bbccdd,fg+:#ddeeff,bg:#334455,preview-bg:#223344,border:#77889    9'"
```

界面配置参数加上后就漂亮多了

`--preview` 表示在右侧显示文件的预览界面，语法高亮的设置使用了 [highlight](https://link.segmentfault.com/?enc=OyAVxaWu0EUGGIk8%2BifWZA%3D%3D.A%2Bk6FOsZXItuAGv8m2jjWznVWXpwo7K8LrUQn9ICn5Of%2BEosXzRBh9U0ENjzvD5ZBOTKzQBibFkosxO5lYjGQw%3D%3D) 如果 highlight 失败则使用最常见的 `cat` 命令来查看文件内容

highlight 安装可能会有个小插曲。highlight 需要手动编译安装，默认安装目录在 `/usr/bin`, `/usr/share` 下面。然而在 macOS 中由于 <abbr title="System Integrity Protection">SIP</abbr> 保护，用户安装的程序不能在这几个目录下面「即使有 sudo 权限也不行」。我们可以手动更改下 highlight 源代码中 makefile 中的参数即可

```makefile
# PREFIX = /usr
PREFIX = /usr/local
```

将 `PREFIX = /usr` 改成 `PREFIX = /usr/local`，然后 `make`，`sudo make install` 就可以了

### 触发命令行补全 FZF_COMPLETION_TRIGGER

默认是 `**`，一般不用修改

## 搜索

fzf 默认会以 “extened-search” 模式启动, 这种模式下不支持正则搜索, 但是你可以输入多个搜索关键词, 以空格分隔, fzf 会无序查找匹配所有字符串.

如 ^music .mp3$, sbtrkt !fire.

fzf 提供了一些增强功能的搜索语法, 如下表所示:

| 标记    | 匹配类型         | 描述                       |
| ------- | ---------------- | -------------------------- |
| sbtrkt  | 模糊匹配         | 内容匹配sbtrkt(字符匹配)   |
| ‘wild   | 精确匹配(单引号) | 内容包含单词wild(单词匹配) |
| ^music  | 前缀精确匹配     | 以music开头                |
| .mp3$   | 后缀精确匹配     | 以.mp3结尾                 |
| !fire   | 反转匹配         | 内容不包含fire             |
| !^music | 前缀反转匹配     | 不以music开头              |
| !.mp3$  | 后缀反转匹配     | 不以.mp3结尾               |

> 注: 如果不想使用模糊匹配或者不想”引用”每个文字, 可以使用 `-e/--exact` 选项. 注意如果使用 `-e/--exact`, 那么 `'` 就变成了解引用, 即:’abc表示匹配a,b和c(a,b,c有序), 而不仅仅是匹配abc.

## 模糊补全

在 bash 或 zsh 终端上, 可以通过输入 `**` 来触发 fzf 对文件/目录的模糊补全(查找), 如下例子所示:

```
# Files under current directory
# - You can select multiple items with TAB key
vim **<TAB>
```

- 进程 ID 模糊补全

在使用kill命令时, fzf 会自动触发其自动补全功能:

```
# Can select multiple processes with <TAB> or <Shift-TAB> keys
kill -9 <TAB>
```

- 主机名补全

如下例子所示:

```
ssh **<TAB>
telnet **<TAB>
```

## 按键绑定

------

fzf 的安装脚本会为 bash, zsh 和 fish 终端设置以下按键绑定:

| 按键   | 描述                           |
| ------ | ------------------------------ |
| CTRL-T | 命令行打印选中内容             |
| CTRL-R | 命令行历史记录搜索, 并打印输出 |
| ALT-C  | 模糊搜索目录, 并进入(cd)       |

# 参考文章

[思否fzf](https://segmentfault.com/a/1190000015977368)

[博客fzf](https://oskernellab.com/2021/02/15/2021/0215-0001-Using_FZF_to_Improve_Productivity/)

# ranger 

## 基础

一个在cli下的文件管理器。

### 安装

```bash
brew install ranger
```

### 图片预览支持，安装`imgcat`

```bash
# 安装
sudo wget -O /usr/local/bin/imgcat https://iterm2.com/utilities/imgcat

#为文件添加权限
sudo chmod 777 /usr/local/bin/imgcat
```

### 生成配置文件（如果第一次使用）

```bash
ranger --copy-config=all
```

将会在`~/.config/ranger`目录输出以下文件

```bash
commands.py          # 与以下命令一起启动的命令 :
commands_full.py    # 全套命令
rc.conf                      # 配置和绑定
rifle.conf                  # 文件关联（用于打开文件的程序）
scope.sh                # 负责各种文件预览
```

修改 `rc.conf`，ranger 将可以直接在终端预览图片

```bash
set preview_images true
set preview_images_method iterm2
```

### 其它相关推荐的依赖安装

```bash
brew install libcaca highlight atool lynx w3m elinks poppler transmission mediainfo exiftool
```

# 快速拷贝目录

* shell的方式

```shell
 echo "$(cd "$(dirname "$1")" && pwd -P)/$(basename "$1")"
```

* mac

```shell
brew install coreutils
greadlink -f file.txt
```



## 集成fzf

ranger是一个终端下的文件浏览器，和它配合使用可以实现文件的寻找并快速跳转。

ranger默认安装完成后没有配置文件，需要执行`ranger --copy-config=all`来生成默认配置文件。文件路径在`~/.config/ranger`。现在可以开始添加配置到*commands.py*，官方的配置你可以在 [这里](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Franger%2Franger%2Fwiki%2FCustom-Commands) 找到

```python

# use fzf in ranger
class fzf_select(Command):
    """
    :fzf_select

    Find a file using fzf.

    With a prefix argument select only directories.

    See: https://github.com/junegunn/fzf
    """
    def execute(self):
        import subprocess
        import os.path
        if self.quantifier:
            # match only directories
            command='fd --full-path  /Users/curiosity/  -H --exclude={.git,.idea,.vscode,.sass-cache,node_modules,build,target,logs,dist,Library,.m2,man,.npm,.node-gyp,Music,Pictures,.nvm,.electron-gyp,.npminstall_tarball,.cache} --type d | fzf +m'
        else:
            # match files and directories
            command='fd --full-path  /Users/curiosity/  -H --exclude={.git,.idea,.vscode,.sass-cache,node_modules,build,target,logs,dist,Library,.m2,man,.npm,.node-gyp,Music,Pictures,.nvm,.electron-gyp,.npminstall_tarball,.cache} --type f | fzf +m'
        fzf = self.fm.execute_command(command, universal_newlines=True, stdout=subprocess.PIPE)
        stdout, stderr = fzf.communicate()
        if fzf.returncode == 0:
            fzf_file = os.path.abspath(stdout.rstrip('\n'))
            if os.path.isdir(fzf_file):
                self.fm.cd(fzf_file)
            else:
                self.fm.select_file(fzf_file)
```

添加完成之后你就可以通过`:fzf_select`命令来在ranger中启动fzf查找，并自动跳转了。当然你可以把这个命令绑定到一个快捷键上，通过在*rc.conf*中添加以下配置。

```shell
# 将fzf查找绑定到ctrl+f键，最好添加到注释Searching后面，我之前添加到前面没有生效，不知道为什么
map <C-f> fzf_select
```

**导航**

- -f =向下翻页
- -b =向上翻页
- gg =转到列表顶部
- G =转到列表底部
- H =返回导航历史记录
- h =移至上级目录
- J =下一页1/2页
- J =下移
- K =上一页1/2页
- k =向上移动
- L =前进浏览历史记录
- Q =退出

**处理文件**

- 我…显示文件
- E | I…编辑文件
- r…使用所选程序打开文件
- cw…重命名文件
- /…搜索文件(n | p跳至下一个/上一个匹配项)
- dd ..将文件标记为剪切
- ud…未切割
- p…粘贴文件
- yy ..复制/粘贴文件
- zh…显示隐藏文件
- =选择当前文件
- ：delete =删除所选文件
- ：mkdir…创建目录
- ：touch…创建一个文件
- ：rename…重命名文件



# IDEA快捷键

## 常用没记住

### 跳转

| 快捷键                   | 说明                                             | 使用频率 |
| :----------------------- | :----------------------------------------------- | :------- |
| ⌘ L                      | 跳转到`指定行`                                   | ★★★★★    |
| ⌘ U                      | 跳转到`父类/接口`的对应处                        | ★★★★★    |
| ⌘ ⌥ B                    | 跳转到`实现`处                                   | ★★★★★    |
| ⌥ (← /→)                 | 光标跳转到当前单词 / 中文句的（左/右）侧开头位置 | ★★★★★    |
| ⌃ ⇧ B                    | 跳转到类`声明`处--在变量上按                     | ★★★      |
| ⌘ ⇧ ⌫                    | 跳转到`最后一次`编辑处                           | ★★       |
| ⌃ ⇧(↓ / ↑)               | 跳转到`上一个` / `下一个`方法名处                | ★★★★★    |
| ⌘⌥  ([ / ])              | 跳转到`当前所在代码块`的花括号`开始`/`结束`处    | ★★       |
| ⌘  (← /→) 或⌃ （ A / E） | 移动光标到**行首** **行尾**                      |          |
| ⌘Fn (← /→)               | 文件首尾                                         |          |
| Fn (↑/↓)                 | 翻页                                             |          |
| ⌘⇧ ⌥  ([ / ])            | 选中到`当前所在代码块`的花括号`开始`/`结束`处    |          |
| ⌃ M                      | 括号匹配，功能同⌘⌥  ([ / ])                      |          |
| ⌥(← /→)                  | 单词移动                                         |          |

### 弹出

| 快捷键       | 说明                                       | 使用频率 |
| :----------- | :----------------------------------------- | :------- |
| ⌘ F12        | 弹出`当前文件`结构，类似eclipse的`outline` | ★★★★★    |
| ⌃ H          | 弹出当前`类`的层次（即父类、子类）         | ★★★★★    |
| ⌘ ⇧ H        | 弹出`方法`层次结构                         | ★★★★★    |
| ⌃ ⌥ H        | 弹出`调用`层次（哪些调用了此处）           | ★★★★★    |
| ⌥ Space, ⌘ Y | 弹出光标所在`方法`、`类`的定义             | ★★★★★    |

### 窗口

| 快捷键  | 说明                                                      | 使用频率 |
| :------ | :-------------------------------------------------------- | :------- |
| ⇧ Esc   | 隐藏`当前`/`最后`一个活动的窗口（且光标进入代码文件窗口） | ★★★★★    |
| F4或⌘ ↓ | `编辑`/`查看`源代码                                       | ★★★★★    |

### 书签

| 快捷键 | 说明                                  | 使用频率 |
| :----- | :------------------------------------ | :------- |
| F3     | 选中文件/文件夹/代码行，添加/取消书签 | ☆        |
| ⌘ F3   | 显示所有书签                          | ☆        |

### 通用

| 快捷键  | 说明                                                         | 使用频率 |
| :------ | :----------------------------------------------------------- | :------- |
| ⌘ ⇧ F12 | 切换最大化编辑器                                             | ★★★★★    |
| ⌃ ⇧ Tab | 编辑窗口标签和工具窗口之间切换（如切换过程中按delete，则关闭对应选中窗口） | ★★★★★    |

### Usage Search（使用地点查询）

| 快捷键     | 说明                                          | 使用频率 |
| :--------- | :-------------------------------------------- | :------- |
| (⌥ / ⌘) F7 | 查找在哪个文件中被使用 / 查找在哪个类中被使用 | ★★★      |
| ⌘ ⇧ F7     | 高亮显示在本文件中使用地点                    | ★★★      |
| ⌘ ⌥ F7     | 显示使用地点                                  | ★★★      |

### 普通edit操作

| 快捷键      | 说明                   | 使用频率 |
| :---------- | :--------------------- | :------- |
| ⌃ ⇧ J       | 智能的将代码拼接成一行 | ☆        |
| ⌘ ↩︎         | 智能的拆分拼接的行     | ☆        |
| ⌘ (+ / -)   | 展开 / 折叠代码块      | ★★★★★    |
| ⌘ ⇧ (+ / -) | 展开 / 折叠所有代码块  | ★★★★★    |

### 显示查看

| 快捷键             | 说明                                                         | 使用频率 |
| :----------------- | :----------------------------------------------------------- | :------- |
| ⌘ P                | 显示方法的参数信息（光标放在被调用方法的圆括号内，然后按此快捷键） | ★★★★     |
| ⌃ J                | 快速显示文档                                                 | ★★★★★    |
| ⇧ F1               | 显示外部文档（在某些代码上会触发打开浏览器显示相关文档）     | ☆        |
| ⌘ + 鼠标放在代码上 | 显示基本信息                                                 | ☆        |
| ⌘ F1               | 在`错误`或`警告`处显示描述信息                               | ☆        |
| ⌃ ⇧ Q              | 显示上下文信息                                               | ☆        |

### 补全

| 快捷键            | 说明                                             | 使用频率 |
| :---------------- | :----------------------------------------------- | :------- |
| ⌃ Space           | 基本的代码补全（补全任何类、方法、变量）         |          |
| ⌃ ⇧ Space         | 智能代码`补全`（过滤器方法列表和变量的预期类型） |          |
| ⌘ ⇧ ↩︎             | 自动结束代码，行末自动添加`分号`                 |          |
| ctrl + option + o | 清除无用 import 快捷键以及自动清除设置           |          |

