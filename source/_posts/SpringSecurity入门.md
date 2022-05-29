---
title: SpringSecurity入门
top: false
cover: false
toc: true
mathjax: true
categories:
  - SpringSecurity
tags:
  - SpringSecurity
date: 2021-04-06 21:45:32
password:
summary:
---
# 一、学习目标
![image-20210406215324351](image-20210406215324351.png)

# 二、OAUTH2 

**微信认证流程**

![微信认证流程](image-20210407212550865.png)

****

![oauth认证流程](image-20210407213041405.png)



## 常用术语

* `客户凭证(client Credentials)`:客户端clientid和密码用与认证客户
* `令牌(tokens)`授权服务器在接收到客户请求后，颁发访问的令牌
* `作用域(scopes):`客户请求访问令牌时，由资源拥有者额外指定细分权限（permission）

## 令牌类型

* `授权码`:仅作用于授权码授权类型，用于交换获取令牌和刷新令牌
* `访问令牌`: 用于代表一个用户，服务直接去访问受保护的资源
* `刷新令牌` :用于去授权服务器获取一个刷新访问令牌
* `BearerToken`:不管谁拿到Token都可以去访问资源，类似于现金
* `Proof of Possession(PoP )Token`: 可以校验client是否对token有明确额拥有权

## 授权模式

### 授权码模式

![授权码模式](image-20210407214427688.png)

### 密码模式

![密码模式](image-20210407214700541.png)

客户端模式

![客户端模式](image-20210407214825939.png)

## 刷新令牌

![刷新令牌](image-20210407214910839.png)

## 授权服务器

![授权服务器](image-20210407215331042.png)

* `Authorize Endpoint` 授权端点，进行授权

* `Token Endpoint ` 令牌端点，经过授权拿到对应的Token
* `Introspection Endpoint` 校验端点，校验Token合法性

* `Revication Endpoint` 撤销端点，撤销授权

## 架构

![image-20210407215819420](image-20210407215819420.png)

### 授权码模式测试

1. 获取授权码

> http://localhost:8080/oauth/authorize?response_type=code&client_id=admin&redirect_uri=www.baidu.com

2. 根据授权码获取token

**携带客户端凭证**

![image-20210407225252378](image-20210407225252378.png)

![image-20210407225222047](image-20210407225222047.png)

```
grant_type:authorization_code
client_id:admin
scope:all
code:NheFN6
redirect_uri:www.baidu.com
```

3. 根据令牌访问资源

![image-20210407225317904](image-20210407225317904.png)

### 密码模式测试

> 密码模式也需要携带client_id

![image-20210407230218565](image-20210407230218565.png)

```
grant_type:password
username:admin
scope:all
password:123456
```

