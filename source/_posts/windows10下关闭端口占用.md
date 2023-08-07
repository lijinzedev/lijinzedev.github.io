---
title: windows10下关闭端口占用
top: false
cover: false
toc: true
mathjax: true
categories:
  - win10
tags:
  - win10
date: 2023-06-13 10:56:05
password:
summary:
---

# windows10下关闭端口占用

## 占用查询端口的pid查询

```css
C:\Users\helloworld>netstat -ano|findstr "9097"
  TCP    0.0.0.0:9097           0.0.0.0:0              LISTENING       6832
  TCP    [::]:9097              [::]:0                 LISTENING       6832
```

## 关闭对应pid

```css
C:\Users\helloworld>taskkill  -F -PID 6832
成功: 已终止 PID 为 6832 的进程。
```

