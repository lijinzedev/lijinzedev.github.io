---
title: 在YAML中使用Maven变量
top: false
cover: false
toc: true
mathjax: true
categories:
  - Maven
tags:
  - Maven
date: 2021-09-02 16:06:42
password:
summary:
---

# 在YAML中使用Maven变量

1. 在外部pom.xml 中定义变量

例如:

```xml
<profiles>
    <!-- 开发环境 -->
    <profile>
        <id>dev</id>
        <activation>
            <!--默认激活配置-->
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <!--当前环境-->
            <pom.profile.name>dev</pom.profile.name>
            <!--Nacos配置中心地址-->
            <pom.nacos.ip>appstore-nacos</pom.nacos.ip>
            <pom.nacos.port>8848</pom.nacos.port>
            <!--Nacos配置中心命名空间,用于支持多环境.这里必须使用ID，不能使用名称,默认为空-->
            <pom.nacos.namespace>mobile</pom.nacos.namespace>
        </properties>
    </profile>
    <!-- 测试环境 -->
    <profile>
        <id>test</id>
        <properties>
            <pom.profile.name>test</pom.profile.name>
            <!--Nacos配置中心地址-->
            <pom.nacos.ip>appstore-nacos</pom.nacos.ip>
            <pom.nacos.port>8848</pom.nacos.port>
            <!--Nacos配置中心命名空间,用于支持多环境.这里必须使用ID，不能使用名称,默认为空-->
            <pom.nacos.namespace>mobile</pom.nacos.namespace>
        </properties>
    </profile>
    <!-- 生产环境 -->
    <profile>
        <id>prod</id>
        <properties>
            <pom.profile.name>prod</pom.profile.name>
            <!--Nacos配置中心地址-->
            <pom.nacos.ip>appstore-nacos</pom.nacos.ip>
            <pom.nacos.port>8848</pom.nacos.port>
            <!--Nacos配置中心命名空间,用于支持多环境.这里必须使用ID，不能使用名称,默认为空-->
            <pom.nacos.namespace>mobile</pom.nacos.namespace>
        </properties>
    </profile>
</profiles>
```

2. 设置[Maven 资源插件](http://maven.apache.org/plugins/maven-resources-plugin/examples/filter.html)以过滤包含**application.yml文件的目录。

```xml
<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
        </resource>
    </resources>
    ...
</build>
```

3. 使用

```yml
appstore:
  nacos:
    ip: ${NACOS_HOST:@pom.nacos.ip@}
    port: ${NACOS_PORT:@pom.nacos.port@}
    namespace: ${NACOS_ID:@pom.nacos.namespace@}
server:
  port: 12000
spring:
  profiles:
    active: @pom.profile.name@
```

