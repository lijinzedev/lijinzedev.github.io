---
title: npm 代理
top: false
cover: false
toc: true
mathjax: true
categories:
  - npm
tags:
  - npm
date: 2021-11-05 10:02:20
password:
summary:
---

# npm设置和取消代理的方法

## 设置代理

```shell
npm config set proxy http://127.0.0.1:7890
npm config set https-proxy http://127.0.0.1:7890
```

如果需要认证的话可以这样设置：

```shell
npm config set proxy http://username:password@server:port
npm confit set https-proxy http://username:password@server:port
```

如果代理不支持https的话需要修改npm存放package的网站地址。

```shell
npm config set registry "http://registry.npmjs.org/"
```

## 使用nrm快速切换npm源

nrm 是一个 NPM 源管理器，允许你快速地在如下 NPM 源间切换：

- 列表项目
- npm
- cnpm
- strongloop
- enropean
- australia
- nodejitsu
- taobao

### Install

```cmake
sudo npm install -g nrm
```

### 如何使用？

列出可用的源：

```awk
  ➜  ~  nrm ls
  npm ---- https://registry.npmjs.org/
  cnpm --- http://r.cnpmjs.org/
  taobao - http://registry.npm.taobao.org/
  eu ----- http://registry.npmjs.eu/
  au ----- http://registry.npmjs.org.au/
  sl ----- http://npm.strongloop.com/
  nj ----- https://registry.nodejitsu.com/
  pt ----- http://registry.npmjs.pt/
```

增加源：

```vim
nrm add <registry> <url> [home]
```

增加源：

```vim
nrm add <registry> <url> [home]
```

测试速度：

```bash
nrm test
```

# 解决Electron安装很慢的办法

```shell
npm config edit
```

该命令会打开npm的配置文件，请在 `registry=https://registry.npm.taobao.org/` 下一行添加

```shell
electron_mirror=https://cdn.npm.taobao.org/dist/electron/ 
```

#### mac系统

```
open .npmrc
```

`npm install electron` 会发现瞬间就安装好了
