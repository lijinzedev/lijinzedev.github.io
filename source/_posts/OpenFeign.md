---
title: OpenFeign
top: false
cover: false
toc: true
mathjax: true
categories:
  - SpringCloud
tags:
  - SpringCloud - OpenFeign
date: 2021-01-23 15:42:33
password:
summary:
---

# 一、概述

[官网](https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/)

> Feign是一个声明式WebService客户端，使用Feign能让编写WebService客户端更加简单

# 二 、使用

## 1 基本使用

* 启动类添加注解`@EnableFeignClients`
* 接口添加注解`@FeignClient`
* 编写对应的Controller

```java
@FeignClient(name = "nacos-order-consumer", path = "/feign")
public interface TestFeign {
    @PostMapping("/get")
    public String providerFeign(@RequestBody Payment payment);
}

```



## 2 文件操作



## 3 超时时间

> 配置OpenFeign的超时控制时间，OpenFeign 内与 ribbon 整合了，支持负载均衡，它的超时控制也由最底层的 ribbon 进行控制，yml 添加配置

```yaml
#设置feign 客户端超时时间（openFeign默认支持ribbon）
ribbon:
  #指的是建立连接后从服务器读取到可用资源所用的时间
  ReadTimeout: 5000
  #指的是建立连接所用的时间，适用于网络状况正常的情况下，两端连接所用的时间
  ConnectTimeout: 5000
```



## 4 日志处理

Feign提供了日志打印功能，我们可以通过配置来调整日志级别，从而了解Feign中Http请求的细节。说白了就是对Feign接口的调用i情况进行监控和输出

**日志级别**

* `None`：默认的，不显示任何日志

* `BASIC`：仅记录请求方法，URL，响应状态码几执行时间

* `HEADERS`：除了BASIC中定义的信息之外，还有请求和响应的头信息

* `FULL`：除了HEADERS中定义的信息之外，还有请求和响应的正文及元数据



```java
@Configuration
public class FeignConfig {

    @Bean
    Logger.Level feignLoggerLevel(){
        return Logger.Level.FULL;
    }
}
```

```yaml
logging:
  level:
    # feign日志以什么级别监控哪个接口
    com.service.PaymentFeignService: debug
```

