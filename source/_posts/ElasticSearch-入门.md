---
title: ElasticSearch 入门
top: false
cover: false
toc: true
mathjax: true
categories:
  - elk
tags:
  - elk
date: 2021-07-15 21:03:54
password:
summary:
---

# 一 ElasticSearch 



### 什么是Elasticsearch?

[官网](https://link.segmentfault.com/?url=https%3A%2F%2Fwww.elastic.co%2Fcn%2Fwhat-is%2Felasticsearch)

> Elasticsearch 是一个分布式的开源搜索和分析引擎，适用于所有类型的数据，包括文本、数字、地理空间、结构化和非结构化数据。Elasticsearch 在 Apache Lucene 的基础上开发而成，由 Elasticsearch N.V.（即现在的 Elastic）于 2010 年首次发布。Elasticsearch 以其简单的 REST 风格 API、分布式特性、速度和可扩展性而闻名，是 Elastic Stack 的核心组件；Elastic Stack 是适用于数据采集、充实、存储、分析和可视化的一组开源工具。人们通常将 Elastic Stack 称为 ELK Stack（代指 Elasticsearch、Logstash 和 Kibana），目前 Elastic Stack 包括一系列丰富的轻量型数据采集代理，这些代理统称为 Beats，可用来向 Elasticsearch 发送数据

着重功能就是用来做**数据的检索和分析**

- **应用程序搜索**
- 网站搜索
- 企业搜索
- **日志处理和分析**
- 基础设施**指标和容器监测**
- 应用程序性能监测
- 地理空间**数据分析和可视化**
- 安全分析
- 业务分析

### mysql也能实现Elasticsearch的功能 为什么还需要Elasticsearch

mysql当然也能实现数据的搜索和分析 例如求年龄的平均值 avg 分组group by 等等 但是我们说术业有专攻，

而mysql专攻于持久化的存储与管理 ，也就是crud 。如果真的使用mysql做海量数据的检索和分析，Elasticsearch更在行，能在秒级给我们响应我们感兴趣的数据，而mysql单表达到百万数据，我们需要进行一些检索和查询都是一些比较慢的操作，比较浪费性能。例如在一些电商系统里面，需要对商品的不同属性按照不同的关键字来检索商品，我们如果用mysql来做，mysql肯定承受不了那么大的压力，而Elasticsearch可以帮我们做到这些。

### 了解ELK

ELK是Elasticsearch、Logstash、Kibana三大开源框架首字母大写简称。市面上也被成为Elastit Stack。其中Elasticsearch是一个基于Lucene、分布式、通过Restful方式进行交互的近实时搜索平台框架。像类似百度、谷歌这种大数据全文搜索引擎的场景都可以使用Elasticsearch作为底层支持框架，可见Elasticsearch提供的搜索能力确实强大,市面上很多时候我们简称Elasticsearch为es。Logstash是ELK的中央数据流引擎，用于从不同目标（文件/数据存储/MQ）收集的不同格式数据，经过过滤后支持输出到不同目的地(文件/MQ/redis/elasticsearch/kafka等 )。Kibana可以将elasticsearch的数据通过友好的页面展示出来，提供实时分析的功能.

市面上很多开发只要提到ELK能够一致说出它是一个日志分析架构技术栈总称，但实际上ELK不仅仅适用于日志分析，它还可以支持其它任何数据分析和收集的场景，日志分析和收集只是更具有代表性。并非唯一性。

![image-20210202103726630](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210715231735.png)

## 1 环境搭建

### 1 编写docker-compose文件

```yaml
version: '3'
services:
  elasticsearch:
    image:  arm64v8/elasticsearch:7.13.3
    container_name: es
    restart: always
    volumes:
      - /home/docker/elasticsearch/data:/usr/share/elasticsearch/data:rw
      - /home/docker/elasticsearch/conf/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
      - /home/docker/elasticsearch/conf/jvm.options:/usr/share/elasticsearch/config/jvm.options
      - /home/docker/elasticsearch/logs:/user/share/elasticsearch/logs:rw
      - /home/docker/elasticsearch/plugins:/usr/share/elasticsearch/plugins
    ports:
        - "9200:9200"
        - "9300:9300"
    environment:
        - discovery.type=single-node
  es-head:
    image: tobias74/elasticsearch-head:6
    container_name: es-head
    restart: always
    ports:
      - "9100:9100"
  kibana:
  	image: arm64v8/kibana:7.13.3
  	container_name: kibana
  	restart: always
	environment:
		- ELASTICSEARCH_URL=http://elasticsearch:9200
    
      
 
```

### 2 编写elasticsearch.yml文件

```yaml
bootstrap.memory_lock: false
cluster.name: "es-server"
node.name: node-1
node.master: true
node.data: true
network.host: 0.0.0.0
http.port: 9200
path.logs: /usr/share/elasticsearch/logs
http.cors.enabled: true
http.cors.allow-origin: "*"
xpack.security.audit.enabled: true
```

### 3 编写jvm配置文件 

```yaml
-Djna.nosys=true

# turn off a JDK optimization that throws away stack traces for common
# exceptions because stack traces are important for debugging
-XX:-OmitStackTraceInFastThrow

# flags to configure Netty
-Dio.netty.noUnsafe=true
-Dio.netty.noKeySetOptimization=true
-Dio.netty.recycler.maxCapacityPerThread=0

# log4j 2
-Dlog4j.shutdownHookEnabled=false
-Dlog4j2.disable.jmx=true

-Djava.io.tmpdir=${ES_TMPDIR}

## heap dumps

# generate a heap dump when an allocation from the Java heap fails
# heap dumps are created in the working directory of the JVM
-XX:+HeapDumpOnOutOfMemoryError

# specify an alternative path for heap dumps; ensure the directory exists and
# has sufficient space
-XX:HeapDumpPath=data

# specify an alternative path for JVM fatal error logs
-XX:ErrorFile=logs/hs_err_pid%p.log

## JDK 8 GC logging

8:-XX:+PrintGCDetails
8:-XX:+PrintGCDateStamps
8:-XX:+PrintTenuringDistribution
8:-XX:+PrintGCApplicationStoppedTime
8:-Xloggc:logs/gc.log
8:-XX:+UseGCLogFileRotation
8:-XX:NumberOfGCLogFiles=32
8:-XX:GCLogFileSize=64m

# JDK 9+ GC logging
9-:-Xlog:gc*,gc+age=trace,safepoint:file=logs/gc.log:utctime,pid,tags:filecount=32,filesize=64m
# due to internationalization enhancements in JDK 9 Elasticsearch need to set the provider to COMPAT otherwise
# time/date parsing will break in an incompatible way for some date patterns and locals
9-:-Djava.locale.providers=COMPAT

# temporary workaround for C2 bug with JDK 10 on hardware with AVX-512
10-:-XX:UseAVX=2
```

# ElasticSearch安装

下载地址:https://www.elastic.co/cn/downloads/elasticsearch

> Windows 下安装

1. 解压

![image-20210202093348097](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210715231645.png)

1. 熟悉目录

   ```bash
   bin 启动文件
   config 配置文件
   	log4j 日志配置文件
   	jvm.options java虚拟机相关的配置
   	ElasticSearch.yml ElasticSearch的配置文件 默认9200端口
   lib 相关jar包
   logs 日志
   modules 功能模块
   plugins 插件
   ```

# 二  核心概念

> ElasticSearch是面向文档的,关系型数据库

| DB               | ElastciSearch |          |
| ---------------- | ------------- | -------- |
| 数据库(datebase) | 索引(indices) |          |
| 表(tables)       | types         | 逐渐弃用 |
| 行(row)          | documents     |          |
| 字段(columns)    | fields        |          |

> elasticesarch 中可以包含多个索引数据库.每个索引中可以包含多个类型(表)每个类型下又包含多个文档(行),每个文档又包含多个字段(列)

**物理设计:**

> elasticsearch 在后台把每个索引划分成多个分片,每分片可以在集群中不同的服务器之间迁移

**逻辑设计:**

一个索引类型中，包含多个文档，比如说文档1，文档2。当我们索引一篇文档时，可以通过这样的一各顺序找到它:索引》类型文档ID，通过这个组合我们就能索引到某个具体的文档。注意:ID不必是整数，实际上它是个字符串

> 文档 

即为一条条数据

之前说elasticsearch是面向文档的，那么就意味着索引和搜索数据的最小单位是文档，elasticsearch中，文档有几个重要属性︰

- 自我包含，一篇文档同时包含字段和对应的值，也就是同时包含key:value !
- 可以是层次型的，一个文档中包含自文档，复杂的逻辑实体就是这么来的!
- 灵活的结构，文档不依赖预先定义的模式，我们知道关系型数据库中，要提前定义字段才能使用，在elasticsearch中，对于字段是非常灵活的，有时候，我们可以忽略该字段，或者动态的添加一个新的字段。

尽管我们可以随意的新增或者忽略某个字段，但是，每个字段的类型非常重要，比如一个年龄字段类型，可以是字符串也可以是整形。因为elasticsearch会保存字段和类型之间的映射及其他的设置。这种映射具体到每个映射的每种类型，这也是为什么在elasticsearch中，类型有时候也称为映射类型。

> 类型

类型是文档的逻辑容器，就像关系型数据库一样，表格是行的容器。类型中对于字段的定义称为映射，比如name映射为字符串类型。我们说文档是无模式的，它们不需要拥有映射中所定义的所有字段，比如新增一个字段，那么elasticsearch是怎么做的呢?elasticsearch会自动的将新字段加入映射，但是这个字段的不确定它是什么类型，elasticsearch就开始猜，如果这个值是18，那么elasticsearch会认为它是整形。但是elasticsearch也可能猜不对，所以最安全的方式就是提前定义好所需要的映射，这点跟关系型数据库殊途同归了，先定义好字段，然后再使用，别整什么幺蛾子。

> 索引

就是数据库

索引是映射类型的容器，elasticsearch中的索引是一个非常大的文档集合。索引存储了映射类型的字段和其他设置。然后它们被存储到了各个分片上了。我们来研究下分片是如何工作的。

**物理设计︰节点和分片如何工作**

![image-20210202134028863](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210715232747.png)

一个集群至少有一个节点，而一个节点就是一个elasricsearch进程，节点可以有多个索引默认的，如果你创建索引，那么索引将会有个5个分片( primary shard ,又称主分片)构成的，每一个主分片会有一个副本( replica shard ,又称复制分片)

![image-20210202134053761](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210715232744.png)

上图是一个有3个节点的集群，可以看到主分片和对应的复制分片都不会在同一个节点内，这样有利于某个节点挂掉了，数据也不至于丢失。实际上，一个分片是一个Lucene索引，一个包含倒排索引的文件目录，倒排索引的结构使得elasticsearch在不扫描全部文档的情况下，就能告诉你哪些文档包含特定的关键字。不过，等等，倒排索引是什么鬼?

> 倒排索引

elasticsearch使用的是一种称为倒排索引的结构，采用Lucene倒排索作为底层。这种结构适用于快速的全文搜索，一个索引文档中所有不重复的列表构成，对于每一个词，都有一个包含它的文档列表。例如，现在有两个文档，每个文档包含如下内容∶

```bash
Study every day,good good up to forever # 文档1包含的内容
To forever, study every day, good good up # 文档2包含的内容
```

为了创建倒排索引，我们首先要将每个文档拆分成独立的词(或称为词条或者tokens)，然后创建一个包含所有不重复的词条的排序列表，然后列出每个词条出现在哪个文档:

![image-20210202134510063](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210715232730.png)

现在，我们试图搜索to forever，只需要查看包含每个词条的文档

![image-20210202134535705](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210715232727.png)

两个文档都匹配，但是第一个文档比第二个匹配程度更高。如果没有别的条件，现在，这两个包含关键字的文档都将返回。再来看一个示例，比如我们通过博客标签来搜索博客文章。那么倒排索引列表就是这样的一个结构:

![image-20210202134608841](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210715232724.png)

如果要搜索含有python标签的文章，那相对于查找所有原始数据而言，查找倒排索引后的数据将会快的多。只需要查看标签这一栏，然后获取相关的文章ID即可。完全过滤掉无关的所有数据，提高效率!

elasticsearch的索引和Lucene的索引对比

在elasticsearch中，索引（库)这个词被频繁使用，这就是术语的使用。在elasticsearch中，索引被分为多个分片，每份分片是一个Lucene的索引。所以一个elasticsearch索引是由多个Lucene索引组成的。别问为什么，谁让elasticsearch使用Lucene作为底层呢!如无特指，说起索引都是指elasticsearch的索引。
