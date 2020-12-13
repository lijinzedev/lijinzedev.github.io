---
title: shell Jar 简单启动脚本
top: false
cover: false
toc: true
mathjax: true
categories:
  - Shell
tags:
  - Shell
date: 2020-12-13 16:40:04
password:
summary:
---

# jar启动脚本

## 启动jar脚本

```bash
pid=$(netstat -nlp | grep :8094 | awk '{print $7}' | awk -F'/' '{print $1}');
if [ -n "$pid" ]; then
    kill -9 $pid;
fi
nohup java -jar -Xmx512m -Xms128m -Xmn64m -Xss64m -Dfile.encoding=UTF-8 /usr/local/testServer/app-operation-1.0-SNAPSHOT.jar >/usr/local/testServer/logs/app-operation.log 2>&1 &

```

## 启动sh脚本

```bash
#!/bin/sh
echo "调用operation";
source   ./app-operation-restart.sh;
```

