---
title: Spring Cacheable + redis
top: false
cover: false
toc: true
mathjax: true
categories:
  - Spring
  - Cacheable
tags:
  - Spring
date: 2020-12-04 11:41:30
password:
summary:
---

# 1 Spring整合Redis缓存

## 1 相关注解

### 1 `@Cacheable`注解

**作用：**

> 主要针对方法配置，能够根据方法的请求参数对结果进行缓存

| 属性      | 解释       | 作用                                                         |
| --------- | ---------- | :----------------------------------------------------------- |
| value     | 缓存的名称 | 每一个缓存名代表一个缓存对象。当一个方法填写多个缓存名称时将创建多个缓存对象。当多个方法使用同一缓存名称时相同的缓存参数会被覆盖。所以通常情况我们使用包名+类名+方法名，或者使用接口的RequestMapping作为缓存名称防止命名重复引起的问题。   <br />单缓存名称：@Cacheable（value=“myccache”）<br />多缓存名称：@Cacheable（value={“cache1”,”cache2”}） |
| key       | 缓存的key  | key标记了缓存对象下的每一条缓存，如果不指定key则系统自动按照方法的所有入参生成key，也就是说相同的入参值会返回同样的缓存结果。<br />如果指定key则要按照SpEL表达式编写使用的入参列表。如下列无论方法存在多少个入参，只要userName值一致，则会返回相同的缓存结果。<br />@Cacheable(value=“test”,key=“#username”) |
| condition | 缓存的条件 | 满足条件后结果才会被缓存。不填写则认为无条件全部缓存<br />条件使用SpEl表达式编写，返回true或者false，只有为true才进行缓存<br />如下例，只有用户名长度大于2时才会进行缓存<br />@Cacheabke(value=“test”,condition=“#username.length()>2”) |

### 2 `@CachePut`注解

> 主要针对方法配置，能够根据方法的请求参数对结果进行缓存、和`@Cacheable`不同的是，它每次都会触发真实方法调用，此注解被常用于更新缓存使用

| 属性      | 解释       | 作用                                                         |
| --------- | ---------- | ------------------------------------------------------------ |
| value     | 缓存的名称 | 例如：<br/>@CachePut(value=”mycache”) <br/>@CachePut(value={”cache1”,”cache2”} |
| key       | 缓存的 key | 例如：<br/>@CachePut(value=”testcache”,key=”#userName”)      |
| condition | 缓存的条件 | 例如：<br/>@CachePut(value=”testcache”,condition=”#userName.length()>2”) |

### 3 `@CacheEvict`注解

> **主要针对方法配置，能够根据一定的条件对缓存进行清空**

| 属性             | 解释                   | 作用                                                         |
| ---------------- | ---------------------- | ------------------------------------------------------------ |
| value            | 缓存的名称             | 删除指定名称的缓存对象。必须与下面的其中一个参数配合使用例如： <br />@CacheEvict(value=”mycache”) 或者 @CacheEvict(value={”cache1”,”cache2”} |
| key              | 缓存的 key             | 删除指定key的缓存对象<br />例如： @CacheEvict(value=”testcache”,key=”#userName”) |
| condition        | 缓存的条件             | 删除指定条件的缓存对象例如： <br />@CacheEvict(value=”testcache”,condition=”#userName.length()>2”) |
| allEntries       | 方法执行后清空所有缓存 | 缺省为 false，如果指定为 true，则方法调用后将立即清空所有缓存。<br />例如：<br /> @CacheEvict(value=”testcache”,allEntries=true) |
| beforeInvocation | 方法执行前清空所有缓存 | 缺省为 false，如果指定为 true，则在方法还没有执行的时候就清空缓存，缺省情况下，如果方法执行抛出异常，则不会清空缓存。<br />例如：<br/>@CacheEvict(value=”testcache”，beforeInvocation=true)<br /> |

## 2 SpringBoot中的Cache

> Spring Boot为我们提供了多种缓存CacheMannger配置方案。默认情况下会使用基于内存map一种缓存方案，
>
> `ConcurrenMapCacheManager`。我们可以通过配置指定 Generic、JCache (JSR-107)、EhCache 2.x、Hazelcast、Infinispan、Redis、Guava、Simple等技术进行缓存实现。

### 1 引入依赖

```properties
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

### 2 启用缓存

```java
@SpringBootApplication 
@EnableCaching //启用缓存
public class App {
    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }
}
```

