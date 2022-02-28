---
title: WebRTC入门
top: false
cover: false
toc: true
mathjax: true
categories:
  - WebRTC
tags:
  - WebRTC
date: 2022-01-24 11:16:02
password:
summary:
---

# WebRtc入门

# [接口文档](https://developer.mozilla.org/zh-CN/docs/Web/API/MediaDevices)



# 连接建立

WebRtc的连接是通过RTCPeerConnection接口可以将本地流MediaStream发送至远端。同时可以将远端媒体流发送至本地，从而建立起对等的连接。



![image-20220125131119425](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202201251311161.png)

提示：图中的“A发送Offer至B”，“B发送Answer至A”，”A发送Candidate至B”，"B发送Candidate至A”等过程并没有通过信令服务器进行中转，而是通过本地变量的方式进行传递。目的是为了简化流程，便于理解。另外Candidate获取也没有架设Sturn服务器。

1. 媒体协商
2. 网络协商
