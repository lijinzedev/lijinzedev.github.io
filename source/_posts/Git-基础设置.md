---
title: Git 代理设置
top: false
cover: false
toc: true
mathjax: true
categories:
  - Git
  - 配置 
tags:
  - Git
date: 2021-07-03 15:11:46
password:
summary:
---

# 仅为 GitHub 设置代理

## git 代理

设置 `git config --global http.https://github.com.proxy socks5://127.0.0.1:7890`
设置完成后, `~/.gitconfig` 文件中会增加以下条目:

```
[http "https://github.com"]
    proxy = socks5://127.0.0.1:7890
```

## ssh 代理

修改 `~/.ssh/config` 文件

```shell
ProxyCommand connect -S 127.0.0.1:7890 -a none %h %p

Host github.com
  User git
  Port 22
  Hostname github.com
  # 注意修改路径为你的路径
  IdentityFile "C:\Users\One\.ssh\id_rsa"
  TCPKeepAlive yes

Host ssh.github.com
  User git
  Port 443
  Hostname ssh.github.com
  # 注意修改路径为你的路径
  IdentityFile "C:\Users\One\.ssh\id_rsa"
  TCPKeepAlive yes
```

# 参考链接

[git ssh 代理设置](https://gist.github.com/chenshengzhi/07e5177b1d97587d5ca0acc0487ad677)

