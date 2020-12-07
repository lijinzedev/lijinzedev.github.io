---
title: wget 设置代理
top: false
cover: false
toc: true
mathjax: true
categories:
  - centos - wget
tags:
  - centos - wget
date: 2020-12-03 16:06:08
password:
summary:

---

```bash
# 永久配置
cd ~/
vim .wgetrc
# 配置
http_proxy = http://your_proxy:port
https_proxy = http://your_proxy:port
proxy_user = user
proxy_password = password
use_proxy = on
wait = 15
```

