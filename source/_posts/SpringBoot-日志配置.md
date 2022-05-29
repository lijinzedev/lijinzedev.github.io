---
title: SpringBoot 日志配置
top: false
cover: false
toc: true
mathjax: true
categories:
  - SpringBoot
  - 日志
tags:
  - SpringBoot
date: 2021-03-31 10:37:12
password:
summary:
---

# SpringBoot 日志配置

> 新建文件logbak.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:noNamespaceSchemaLocation="http://www.padual.com/java/logback.xsd"
    debug="false" scan="true" scanPeriod="30 second">
	<!--读取配置中心的属性-->
    <springProperty scope="context" name="name" source="spring.application.name"/>
    <property name="ROOT" value="/opt/logs/hxkf/resource/" />
    <property name="FILESIZE" value="10MB" />
    <property name="MAXHISTORY" value="60" />
    <property name="DATETIME" value="yyyy-MM-dd HH:mm:ss" />
    <!-- 控制台打印 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder charset="utf-8">
            <pattern>[%-5level] %d{${DATETIME}} [%thread] %logger{36} - %m%n
            </pattern>
        </encoder>
    </appender>
    <!-- ERROR 输入到文件，按日期和文件大小 -->
    <appender name="ERROR" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <encoder charset="utf-8">
            <pattern>[%-5level] %d{${DATETIME}} [%thread] %logger{36} - %m%n
            </pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${ROOT}%d/error.%i.log</fileNamePattern>
            <maxHistory>${MAXHISTORY}</maxHistory>
            <timeBasedFileNamingAndTriggeringPolicy
                class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>${FILESIZE}</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
    </appender>
    
    <!-- WARN 输入到文件，按日期和文件大小 -->
    <appender name="WARN" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <encoder charset="utf-8">
            <pattern>[%-5level] %d{${DATETIME}} [%thread] %logger{36} - %m%n
            </pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>WARN</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${ROOT}%d/warn.%i.log</fileNamePattern>
            <maxHistory>${MAXHISTORY}</maxHistory>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>${FILESIZE}</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
    </appender>
    
    <!-- INFO 输入到文件，按日期和文件大小 -->
    <appender name="INFO" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <encoder charset="utf-8">
            <pattern>[%-5level] %d{${DATETIME}} [%thread] %logger{36} - %m%n
            </pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>INFO</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${ROOT}%d/info.%i.log</fileNamePattern>
            <maxHistory>${MAXHISTORY}</maxHistory>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>${FILESIZE}</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
    </appender>
    <!-- DEBUG 输入到文件，按日期和文件大小 -->
    <appender name="DEBUG" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <encoder charset="utf-8">
            <pattern>[%-5level] %d{${DATETIME}} [%thread] %logger{36} - %m%n
            </pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>DEBUG</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${ROOT}%d/debug.%i.log</fileNamePattern>
            <maxHistory>${MAXHISTORY}</maxHistory>
            <timeBasedFileNamingAndTriggeringPolicy
                class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>${FILESIZE}</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
    </appender>
    <!-- TRACE 输入到文件，按日期和文件大小 -->
    <appender name="TRACE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <encoder charset="utf-8">
            <pattern>[%-5level] %d{${DATETIME}} [%thread] %logger{36} - %m%n
            </pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>TRACE</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        <rollingPolicy
            class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${ROOT}%d/trace.%i.log</fileNamePattern>
            <maxHistory>${MAXHISTORY}</maxHistory>
            <timeBasedFileNamingAndTriggeringPolicy
                class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>${FILESIZE}</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
    </appender>
    
    <!-- SQL相关日志输出-->
    <logger name="org.mybatis.spring" level="DEBUG" additivity="true" />
    
    <!-- Logger 根目录 -->
    <root level="INFO">
        <appender-ref ref="STDOUT" />
        <appender-ref ref="DEBUG" />  
        <appender-ref ref="ERROR" />
        <appender-ref ref="WARN" />
        <appender-ref ref="INFO" /> 
        <appender-ref ref="TRACE" />
    </root>
</configuration>
```

