---
title: GateWay
top: false
cover: false
toc: true
mathjax: true
categories:
  - Gateway
tags:
  - SpringCloud - Gateway
date: 2021-01-24 16:50:13
password:
summary:
---

# 一、概念

[官网](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/)

> GateWay是在Spring生态系统之上构建的API网关服务，基于Spring5，SpringBoot2，和Project Reactor等技术，Gateway只在提供一种简单而有效的方式来对API进行路由，以及提供一些强大的过滤器功能，；例如，熔断、限流、重试等

> SpringCloud Gateway作为Spring Cloud生态系统中的网关，目标是替代Zuul，在SpringCloud 2.0以上版本中，没有对新版本的Zuul2.0以上最新高性能版本进行集成，仍然使用Zuul1.x非Reactor模式的老版本，为了提升网关的性能，SpringCloud Gateway是基于WebFlux框架实现的，而WebFlux框架底层则使用了高性能的Reactor模式通信框架Netty

> SpringCloud GateWay的目标提供统一的路由方式且基于Filter链的方式提供了网关的基本功能，例如：安全，监控、指标、和限流

![image-20210124170828606](image-20210124170828606.png)

## 1 路由

> 路由是构建网关的基本模块，它由ID，目标URI，一系列的断言和过滤器组成，如果断言为true则匹配该路由

## 2 Predicate（断言）

> 参考的是java8的java.util.function.Predicate开发人员可以匹配HTTP请求中的所有内容（例如请求头或请求参数），如果请求与断言相匹配则进行路由

## 3 Filter(过滤)

> 指的是Spring框架中GatewayFilter的实例，使用过滤器，可以在请求被路由前或者之后对请求进行修改。

## 4 核心逻辑

> 路由转发+执行过滤器链

## 5 总结

![image-20210124184814535](image-20210124184814535.png)

> 客户端向SpringCloud Gatway发出请求。然后在GateWay Handler Mapping中找到与请求相匹配的路由，将其发送到GateWay Web Handler上
>
> Handler再通过指定的过滤器链来将请求发送到我们实际的服务执行业务逻辑，然后返回，过滤器之间用虚线分开是因为过滤器可能会在发送代理请求之前或之后执行业务    逻辑
>
> Filter在pre类型的过滤器可以做参数校验、权限校验、流量监控、日志输出、协议转换等，在post类型的过滤器可以做响应内容、响应头修改、日志输出、流量监控等

![image-20210124184847772](image-20210124184847772.png)

# 二、使用

## 1 配置

**POM依赖**

> 注意：因为GateWay是WebFlux要剔除SpringWeb

```xml
<!--新增gateway-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

**YAML文件**

> 默认情况下Gateway会根据注册中心的服务列表，以注册中心上微服务名为路径创建动态路由进行转发，从而实现动态路由的功能

```yaml
server:
  port: 9527
spring:
  application:
    name: cloud-gateway
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
    gateway:
      routes:
        - id: payment_routh #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://nacos-order-consumer   #匹配后提供服务的路由地址
          predicates:
            - Path=/order/feign/**   #断言,路径相匹配的进行路由
          #StripPrefix=1就代表截取路径的个数，这样配置后当请求/config/feign/get后端匹配到的请求路径，
          #就会变成http://localhost:8762/feign/get后端匹配到的请求路径
          filters:
            - StripPrefix=1
        - id: payment_routh2
          uri: lb://nacos-config-client

          predicates:
            - Path=/config/feign/**   #断言,路径相匹配的进行路由
        #StripPrefix=1就代表截取路径的个数，这样配置后当请求/config/feign/get后端匹配到的请求路径，
        #就会变成http://localhost:8762/feign/get后端匹配到的请求路径
          filters:
            - StripPrefix=1

```

* 需要注意的是uri的协议为lb，表示启用Gateway的负载均衡功能。
* lb://serviceName是spring cloud gateway在微服务中自动为我们创建的负载均衡uri

![Route Predicate Factories](image-20210124205119110.png)

## 2 代码实现配置

- 代码实现

```java
@Bean
public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
                .route(r -> r.path("/activity/**")
                        .filters(f -> f.stripPrefix(1).filter(new TestGetWayFilter()).addResponseHeader("X-Response-Default-Foo", "Default-Bar"))
                        .uri("lb://activity")
                        .order(0)
                        .id("activity-route")
                )
                .build();
 }
```





## 3 常用的Route Predicate

* `After` Route Predicate

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - After=2017-01-20T17:42:47.789-07:00[America/Denver]
```

*  `Before` route predicate

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: before_route
        uri: https://example.org
        predicates:
        - Before=2017-01-20T17:42:47.789-07:00[America/Denver]
```

* `Between` route predicate

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: between_route
        uri: https://example.org
        predicates:
        - Between=2017-01-20T17:42:47.789-07:00[America/Denver], 2017-01-21T17:42:47.789-07:00[America/Denver]
```

* `Cookie` route predicate

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: cookie_route
        uri: https://example.org
        predicates:
        - Cookie=chocolate, ch.p
```

This route matches requests that have a cookie named `chocolate` whose value matches the `ch.p` regular expression.

* `Header` route predicate

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: header_route
        uri: https://example.org
        predicates:
        - Header=X-Request-Id, \d+
```

This route matches if the request has a header named `X-Request-Id` whose value matches the `\d+` regular expression (that is, it has a value of one or more digits).

* `Host` route predicate

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: https://example.org
        predicates:
        - Host=**.somehost.org,**.anotherhost.org
```

[详见官网](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gateway-request-predicates-factories)

# 三 、过滤器

> Route filters allow the modification of the incoming HTTP request or outgoing HTTP response in some manner. Route filters are scoped to a particular route. Spring Cloud Gateway includes many built-in GatewayFilter Factories.
>
> 路由过滤器可用于修改进入HTTP请求和返回的HTTP响应，路由过滤器智能指定路由进行使用，SPringCloudGateWay内置了多种路由过滤器，他们由GateWayFilter的工厂类来产生

## 1 常用的GatewayFilter

* `AddRequestParameter`

> This listing adds `X-Request-red:blue` header to the downstream request’s headers for all matching requests.
>
> `AddRequestHeader` is aware of the URI variables used to match a path or host. URI variables may be used in the value and are expanded at runtime. The following example configures an `AddRequestHeader` `GatewayFilter` that uses a variable:
>
> 会在匹配的请求头上加一对请求头

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_header_route
        uri: https://example.org
        filters:
        - AddRequestHeader=X-Request-red, blue
```

* [其余见官网](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gatewayfilter-factories)

## 2 自定义过滤器

### 1 实现接口

* GlobalFilter 
* Ordered

### 2 作用

* 全局日志记录

* 统一网关鉴权等

  