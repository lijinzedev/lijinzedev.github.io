---
title: Devtools
top: false
cover: false
toc: true
mathjax: true
categories:
  - Idea
tags:
  - Idea
date: 2021-01-16 23:07:19
password:
summary:
---

# 1 添加依赖

```xml
<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-devtools</artifactId>
	<scope>runtime</scope>
<optional>true</optional>
```

# 2 root工程添加

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <fork>true</fork>
                <addResources>true</addResources>
            </configuration>
        </plugin>
    </plugins>
</build>
```

# 3 开启自动构建

![开启自动构建](image-20210116231753206.png)

# 4 Update the value of

快捷键`ctrl+shift+alt+/`

![开启](image-20210116232054413.png)

![开启](image-20210116232159614.png)