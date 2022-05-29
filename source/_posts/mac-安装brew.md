---
title: mac 安装brew
top: false
cover: false
toc: true
mathjax: true
categories:
  - Mac
  - 环境搭建
tags:
  - Mac
date: 2021-09-14 15:16:12
password:
summary:
---

# brew安装

1. 安装SwitchHost
3. 改host，一定要用强大的且齐全的host
4. 终端自己改代理

**终端如何改代理**
这个10080要改成你自己的

```bashba s
export https_proxy=http://127.0.0.1:10080 http_proxy=http://127.0.0.1:10080 all_proxy=socks5://127.0.0.1:10080
```

**官网给出的安装命令行**

安装：

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```


卸载：

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/uninstall.sh)"
```

**国内一键安装地址**

```bash
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
```

