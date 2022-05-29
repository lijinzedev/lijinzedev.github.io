---
title: Nacos入门
top: false
cover: false
toc: true
mathjax: true
categories:
  - SpringCloud
  - Nacos 
tags:
  - SpringCloud 
date: 2021-01-15 21:52:10
password:
summary:
---

​                                                                                                                                                                                                                                                                                                                                                                                                                                                                      

# 一 、简介

## 1 为什么叫Nacos

> 前四个字母分贝为Naming和Configuration的前两个字母，最后的s为Service

## 2 Nacos是什么

1. 易于构建云原生应用的动态服务发现，配置管理和服务管理中心
2. Nacos：Dynamic Naming and Configuration Service
3. Nacos就是注册中心与配置中心的组合

## 3 Ncaos作用

1. 替代Eureka做服务注册中心
2. 替代Config做服务配置中心
3. **Nacos** = `Eureka`+`Config`+`Bus`

## 4 Nacos官网

* [官网](https://github.com/alibaba/Nacos)

* [官方文档](https://nacos.io/zh-cn/index.html)
* [官方文档GitHub](https://spring-cloud-alibaba-group.github.io/github-pages/greenwich/spring-cloud-alibaba.html#_spring_cloud_alibaba_nacos_discovery)

## 5 Nacos比较

| 服务注册与发现框架 | CAP模型 | 控制台管理 | 社区活跃度      |
| ------------------ | ------- | ---------- | --------------- |
| Eureka             | AP      | 支持       | 低(2.x版本闭源) |
| Zookeeper          | CP      | 不支持     | 中              |
| Consul             | CP      | 支持       | 高              |
| Nacos              | AP      | 支持       | 高              |

## 6 Nacos的三种部署模式

```bash
# docker 启动
docker run --name nacos -e MODE=standalone -p 8848:8848 -d nacos/nacos-server:latest
```



### 1 单机模式

> 单机模式：可用于测试和单机使用，生产环境切忌使用单机模式（满足不了高可用）

```bash
# linux
./startup.sh -m standalone
# Windows
startup.cmd -m standalone
```

### 2 集群模式

> 集群模式：可用于生产环境，确保高可用

```bash
# linux
./startup.sh -m cluster
# Windows
startup.cmd -m cluster
```



### 3 多集群模式

> 多集群模式：可用于多数据中心场景



# 二、注册中心入门

![自带负载均衡](image-20210117164440317.png)

## **1 父工程pom添加**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-dependencies</artifactId>
    <version>Hoxton.SR1</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

## **2 本工程pom**

```xml
<dependencies>
    <dependency>
        <groupId>com.stu</groupId>
        <artifactId>commons</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.62</version>
    </dependency>
</dependencies>
```

## **3 application.yml文件**

```yaml
server:
  port: 9001
spring:
  application:
    name: nacos-payment-provider
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

## **4 启动类**

```java
@EnableDiscoveryClient
@SpringBootApplication
public class OrderNacosMain83
{
    public static void main(String[] args)
    {
        SpringApplication.run(OrderNacosMain83.class,args);
    }
}
```

# 三、服务注册中心对比

## 1 Nacos 全景图所示

![全景图](image-20210117170026388.png)

## 2 Nacos和CAP

![Nacos和CAP](image-20210117170106013.png)

![Nacos服务发现实例模型](image-20210117170124102.png)

## 3 切换

> C是所有节点在同一时间看到的数据是一致的，而A的定义是所有的请求都会收到响应

**何时选择何种模式**

> 一般来说，如果不需要存储服务级别的信息且服务实例是通过nacos-client注册，并且能够保持信条上报，那么就可以选择AP模式，当前主流的服务如Spring Cloud和Dobbo服务用于AP模式，AP模式为了服务的可能性而减弱了一致性，因此AP模式下只支持注册临时实例
>
> 如果需要在服务级别编辑存储配置信息，那么CP是必须，K8S服务和DNS服务则适用于CP模式
>
> CP模式下则支持注册持久化实例，此时则是以Raft协议为集群运行模式，该模式注册实例之前必须要先注册服务，如果服务不存在，则返回错误

```bash
curl -X PUT '$NACOS_SERVER:8848/nacos/v1/ns/operator/switches?entry=serverMode&value=CP'
```

# 四、服务配置中心入门

## 1 POM文件

```xml
<dependencies>
    <!--nacos-config-->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
    </dependency>
    <!--nacos-discovery-->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    <!--web + actuator-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <!--一般基础配置-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>

```

## 2 YAML

**为什么配置两个**?

> Nacos同SpringCloud-Config一样，在项目初始化时，要保证先从配置中心进行配置拉取，拉取配置之后，才能保证项目的正常启动

> SpringBoot中配置文件的家在是存在优先级顺序的，bootstrap优先级高于application

### 1 application.yml

> 个性化配置

```yaml
spring:
  profiles:
    active: dev
```

### 2 bootstrap.yml

> 全局配置

```yaml
server:
  port: 3377
spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #服务注册中心地址
      config:
        server-addr: localhost:8848 #配置中心地址
        file-extension: yaml #指定yaml格式的配置
        
DataId为的基本配置${spring.application.name}-${spring.profiles.active}. ${spring.cloud.nacos.config.file-extension}。
```

## 3 启动类

```java
@EnableDiscoveryClient
@SpringBootApplication
public class NacosConfigClientMain3377
{
    public static void main(String[] args) {
        SpringApplication.run(NacosConfigClientMain3377.class, args);
    }
}
```

## 4 业务

```java
@RestController
/**
 * SpringCloud 实现配置自动刷新
 */
@RefreshScope
public class ConfigClientController {
    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/config/info")
    public String getConfigInfo() {
        return configInfo;
    }
}
 
```

## 5 Nacos添加配置中心

[官方文档](https://nacos.io/zh-cn/docs/quick-start-spring-cloud.html)

### 1 公式

* prefix默认为spring.application.name的值
* spring.profile.active既为当前环境对应的profile,可以通过配置项spring.profile.active 
* file-exetension为配置内容的数据格式，可以通过配置项spring.cloud.nacos.config.file-extension配置

```bash
${spring.application.name}-${spring.profile.active}.${spring.cloud.nacos.config.file-extension}
```

# 五、Nacos作为配置中心-分类配置

## 1 问题

实际开发中，通常一个系统会准备

* dev开发环境
* test测试环境
* prod生产环境

**问题一**：

如何保证指定环境启动时服务能正常读取到Nacos上相应的环境配置文件呢?

**问题二**

一个大型分布式无服务系统会有很多服务的子项目，每个微服务项目又都会有对应的开发环境，测试环境，预发环境，正式环境，那么怎么对这些微服务进行管理呢？

## 2 图形界面

> Nacos图形界面中有配置管理与命名了空间

## 3 Namespace+Group+Data ID三者关系？为什么这么设计？

![image-20210117181600858](image-20210117181600858.png)

![image-20210117181714570](image-20210117181714570.png)

## 4 Nacos之DataID配置

* 指定spring.profile.active和配置文件的DataID来使不同环境下读取不同的配置
* 默认空间+默认分组+新建dev和test两个DataID
* 通过spring.profile.active属性就能进行多环境下配置文件的读取

## 5 Nacos之Group配置

* 在nacos图形界面控制台上面新建配置文件DataID
* bootstrap添加Group为对应的Group名称



## 6 Nacos之Namespace

* 新建Namespace
* 在bootstrap中添加对应ID

# 六、Nacos集群与持久化配置

## 1 概念

[官网](https://nacos.io/zh-cn/docs/cluster-mode-quick-start.html)

![架构](image-20210117184851414.png)

![图片来自尚硅谷](image-20210117185056772.png)

### 重点说明

![重点说明](image-20210117185447278.png)

![重点说明](image-20210117185550557.png)

## 2 Nacos持久化配置

默认嵌入式数据库derby

[官网](https://github.com/alibaba/nacos/blob/develop/config/pom.xml)

###  derby切换到mysql

1. nacos-server-1.1.4\nacos\conf目录下找到sql脚本
   * `nacos-mysql.sql mysql脚本`
   * `schema.sql  derby脚本`
2. 运行nacos-mysql.sql脚本
3. 修改application.properties配置

```properties
db.num=1
db.url.0=jdbc:mysql://localhost:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=root
db.password=123456
```

## 3 Nacos 集群安装

![架构图来自尚硅谷](image-20210117213444032.png)

### 1 环境搭建

[下载地址](https://github.com/alibaba/nacos/releases/tag/1.1.4)

* Nginx -  1 个
* Nacos - 3 个
* Mysql  - 1 个

### 2 配置

#### 1  普通版

1. 梳理出三台nacos机器的不同服务端口号
2. 编辑cluster.conf

```bash
192.168.0.1:1111
192.168.0.2:2222
192.168.0.3:3333
```

3. 编辑stratup.sh

> 添加 -p 参数用于指定Nacos启动的端口号

![图片来自尚硅谷](image-20210117212216016.png)

![图片来自尚硅谷](image-20210117212241094.png)

4. 执行

```bash
./stratup.sh -p 1111
./stratup.sh -p 2222
./stratup.sh -p 3333
```

5. Nginx配置

```
upstream cluster{                                                 
    server 192.168.0.1:1111;
    server 192.168.0.2:2222;
    server 192.168.0.3:3333;
}
server{
                          
    listen 8888;
    server_name localhost;
    location /{
         proxy_pass http://cluster;                                                     
    }
```

```bash
# 启动
./nginx -c /usr/local/nginx/conf/nginx/conf
```

#### 2 Docker 版

1. 数据库以及配置文件修改`略`

2. 创建并运行Nacos容器

**获取镜像**

```bash
docker pull nacos/nacos-server
```

**启动Nacos1111**

```bash
docker run -d \
-e PREFER_HOST_MODE=hostname \
-e MODE=cluster \
-e NACOS_APPLICATION_PORT=8848 \
-e NACOS_SERVERS="192.168.0.1:1111 192.168.0.1:2222 192.168.0.1:3333" \
-e SPRING_DATASOURCE_PLATFORM=mysql \
-e MYSQL_SERVICE_HOST=192.168.0.100 \
-e MYSQL_SERVICE_PORT=3306 \
-e MYSQL_SERVICE_USER=root \
-e MYSQL_SERVICE_PASSWORD=123456 \
-e MYSQL_SERVICE_DB_NAME=nacos-config \
-e NACOS_SERVER_IP=192.168.0.1 \
-p 1111:8846 \
--name nacos1111 \
nacos/nacos-server
```

**启动Nacos2222**

```bash
docker run -d \
-e PREFER_HOST_MODE=hostname \
-e MODE=cluster \
-e NACOS_APPLICATION_PORT=8848 \
-e NACOS_SERVERS="192.168.0.1:1111 192.168.0.1:2222 192.168.0.1:3333" \
-e SPRING_DATASOURCE_PLATFORM=mysql \
-e MYSQL_SERVICE_HOST=192.168.0.100 \
-e MYSQL_SERVICE_PORT=3306 \
-e MYSQL_SERVICE_USER=root \
-e MYSQL_SERVICE_PASSWORD=123456 \
-e MYSQL_SERVICE_DB_NAME=nacos-config \
-e NACOS_SERVER_IP=192.168.0.1 \
-p 2222:8846 \
--name nacos2222 \
nacos/nacos-server
```

**启动Nacos3333**

```bash
docker run -d \
-e PREFER_HOST_MODE=hostname \
-e MODE=cluster \
-e NACOS_APPLICATION_PORT=8848 \
-e NACOS_SERVERS="192.168.0.1:1111 192.168.0.1:2222 192.168.0.1:3333" \
-e SPRING_DATASOURCE_PLATFORM=mysql \
-e MYSQL_SERVICE_HOST=192.168.0.100 \
-e MYSQL_SERVICE_PORT=3306 \
-e MYSQL_SERVICE_USER=root \
-e MYSQL_SERVICE_PASSWORD=123456 \
-e MYSQL_SERVICE_DB_NAME=nacos-config \
-e NACOS_SERVER_IP=192.168.0.1 \
-p 3333:8846 \
--name nacos3333 \
nacos/nacos-server
```

**配置Nginx**

```
	upstream cluster{
        server 192.168.248.129:8846;
        server 192.168.248.129:8847;
        server 192.168.248.129:8848;
    }
    server {
        listen 8080;
        server_name _;

        location / {
            proxy_pass http://cluster;
        }
    }

```

**启动Nginx**

```bash
docker run --name my-nginx -v /usr/local/nginx/conf/nginx.conf:/etc/nginx/nginx.conf:ro -p 8080:8080 -d nginx
```

# 七、Nacos微服务注册地址为内网IP的解决办法

Nacos注册中心是: https://github.com/alibaba/nacos

各个服务通过Nacos客户端将服务信息注册到Nacos上
当Nacos服务注册的IP默认选择出问题时，可以通过查阅对应的客户端文档，来选择配置不同的网卡或者IP
`（参考org.springframework.cloud.alibaba.nacos.NacosDiscoveryProperties的配置）`

例如，使用了Spring cloud alibaba（官方文档）作为Nacos客户端，
服务默认获取了内网IP `192.168.1.21`,
可以通过配置`spring.cloud.inetutils.preferred-networks=10.34.12`，使服务获取内网中前缀为`10.34.12`的IP

解决方法：

1、直接添加忽略某张网卡的配置：

```properties
spring.cloud.inetutils.ignored-interfaces[0]=eth0 # 忽略eth0, 支持正则表达式
```

正则：

```properties
spring.cloud.inetutils.ignored-interfaces=eth.*
```

2、指定默认IP：

```properties
spring.cloud.inetutils.preferred-networks=192.168.20.123 #可以是IP段：192.168.20
```

3、除了这些配置，还有以下的这些配置：

```properties
# 如果选择固定Ip注册可以配置
spring.cloud.nacos.discovery.ip = 10.2.11.11
spring.cloud.nacos.discovery.port = 9090

# 如果选择固定网卡配置项
spring.cloud.nacos.discovery.networkInterface = eth0

# 如果想更丰富的选择，可以使用spring cloud 的工具 InetUtils进行配置
# 具体说明可以自行检索: https://github.com/spring-cloud/spring-cloud-commons/blob/master/docs/src/main/asciidoc/spring-cloud-commons.adoc
spring.cloud.inetutils.default-hostname
spring.cloud.inetutils.default-ip-address
spring.cloud.inetutils.ignored-interfaces[0]=eth0 	# 忽略网卡，eth0
spring.cloud.inetutils.ignored-interfaces=eth.* 	# 忽略网卡，eth.*，正则表达式
spring.cloud.inetutils.preferred-networks=10.34.12 	# 选择符合前缀的IP作为服务注册IP
spring.cloud.inetutils.timeout-seconds
spring.cloud.inetutils.use-only-site-local-interfaces
spring.cloud.nacos.discovery.server-addr  #Nacos Server 启动监听的ip地址和端口
spring.cloud.nacos.discovery.service  #给当前的服务命名
spring.cloud.nacos.discovery.weight  #取值范围 1 到 100，数值越大，权重越大
spring.cloud.nacos.discovery.network-interface #当IP未配置时，注册的IP为此网卡所对应的IP地址，如果此项也未配置，则默认取第一块网卡的地址
spring.cloud.nacos.discovery.ip  # 优先级最高
spring.cloud.nacos.discovery.port  # 默认情况下不用配置，会自动探测
spring.cloud.nacos.discovery.namespace # 常用场景之一是不同环境的注册的区分隔离，例如开发测试环境和生产环境的资源（如配置、服务）隔离等。

spring.cloud.nacos.discovery.access-key  # 当要上阿里云时，阿里云上面的一个云账号名
spring.cloud.nacos.discovery.secret-key # 当要上阿里云时，阿里云上面的一个云账号密码
spring.cloud.nacos.discovery.metadata    #使用Map格式配置，用户可以根据自己的需要自定义一些和服务相关的元数据信息
spring.cloud.nacos.discovery.log-name   # 日志文件名
spring.cloud.nacos.discovery.enpoint   # 地域的某个服务的入口域名，通过此域名可以动态地拿到服务端地址
ribbon.nacos.enabled  # 是否集成Ribbon 默认为true

```

> ignored-interfaces和preferred-networks这两个配置。这两个配置决定了spring cloud应用在启动的时候所使用的网卡和IP地址。ignored-interfaces接收一个正则表达式数组，配置名字虽然是ignored-interfaces，忽略的网卡，但是因为其接收的是正则表达式，所以我们可以任意的选择和反选本机的网卡。preferred-networks是指倾向于使用的IP地址，接收一个正则表达式数组，用于选择Spring Cloud应用使用的本机的IP地址。通过这两个配置，我们可以任意指定Spring Cloud应用使用的网卡和IP地址。
>
> 更多解释参考官方说明，[spring-cloud-commons](https://github.com/spring-cloud/spring-cloud-commons/blob/master/docs/src/main/asciidoc/spring-cloud-commons.adoc)项目为Spring Cloud生态提供了顶层的抽象和基础设施的实现。 网络这个最基本的基础设施也是在这里有对应的实现：InetUtils、InetUtilsProperties和UtilAutoConfiguration提供了网络配置相关的功能。