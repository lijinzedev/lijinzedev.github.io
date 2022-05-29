---
title: ElasticSearch 入门
top: false
cover: false
toc: true
mathjax: true
categories:
  - 中间件 
  - elk
tags:
  - 中间件
  - elk
date: 2021-07-15 21:03:54
password:
summary:
---

环境:

* 树莓派4b ubuntu 20
* es 7.13.2

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
    image:  arm64v8/elasticsearch:7.13.2
    container_name: es
    restart: always
    volumes:
      - /home/docker/elasticsearch/data:/usr/share/elasticsearch/data:rw
      - /home/docker/elasticsearch/conf/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
      - /home/docker/elasticsearch/logs:/user/share/elasticsearch/logs:rw
      - /home/docker/elasticsearch/plugins:/usr/share/elasticsearch/plugins
    ports:
        - "9200:9200"
        - "9300:9300"
    environment:
        - discovery.type=single-node
  es-head:
    image: aichenk/elasticsearch-head:5-alpine
    container_name: es-head
    restart: always
    ports:
      - "9100:9100"
  kibana:
    image: arm64v8/kibana:7.13.2
    container_name: kibana
    restart: always
    environment:
      - ELASTICSEARCH_URL=http://elasticsearch:9200
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch
 
```

- **注意**：宿主机的目录需要赋权，否则启动会报 **failed to bind service AccessDeniedException** 错误

  ```bash
  chmod /home/docker/elasticsearch/data
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

### 3 汉化kibana

```bash
# 查看目录下是否又中文json
ls /usr/share/kibana/x-pack/plugins/translations/translations
# 更改配置文件
cd /usr/share/kibana/config
vi kibana.yml
添加i18n.locale: "zh-CN"
# 重启容器
docker-compose restart kibana
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

## 1 ElasticSearch相关的数据类型

- 字符串类型：text、keyword
- 数值类型：long、integer、short、byte、doule、float、half float、scaled float
- 日期类型：date
- 布尔值类型：boolean
- 二进制类型：binary
- 等等......



# 三 IK分词器

分词：即把一段中文或者别的划分成一个个的关键字，我们在搜索时候会把自己的信息进行分词，会把数据库中或者索引库中的数据进行分词，然后进行一个匹配操作，默认的中文分词是将每个字看成一个词，这显然是不符合要求的，所以我们需要安装中文分词器IK来解决这个问题。

**使用中文，建议使用IK分词器！**

## 1 [安装](https://github.com/medcl/elasticsearch-analysis-ik)

download or compile

- optional 1 - download pre-build package from here: https://github.com/medcl/elasticsearch-analysis-ik/releases

  create plugin folder `cd your-es-root/plugins/ && mkdir ik`

  unzip plugin to folder `your-es-root/plugins/ik`

```bash
# 查看插件
docker exec elasticsearch elasticsearch-plugin list
```

## 2 测试

* ik_smart 最少切分

![image-20210717005956619](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210717005958.png)

* ik_max_word为最细粒度划分，字典！穷尽所有可能

![image-20210717010023634](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210717010025.png)

## 3 IK分词器增加自己的配置（保存后，需要重启）

> IKAnalyzer.cfg.xml` can be located at `{conf}/analysis-ik/config/IKAnalyzer.cfg.xml` or `{plugins}/elasticsearch-analysis-ik-*/config/IKAnalyzer.cfg.xml

在ik的config目录下添加新的字典文件

```xml

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
	<comment>IK Analyzer 扩展配置</comment>
	<!--用户可以在这里配置自己的扩展字典 -->
	<entry key="ext_dict">custom/mydict.dic;custom/single_word_low_freq.dic</entry>
	 <!--用户可以在这里配置自己的扩展停止词字典-->
	<entry key="ext_stopwords">custom/ext_stopword.dic</entry>
 	<!--用户可以在这里配置远程扩展字典 -->
	<entry key="remote_ext_dict">location</entry>
 	<!--用户可以在这里配置远程扩展停止词字典-->
	<entry key="remote_ext_stopwords">http://xxx.com/xxx.dic</entry>
</properties>
```

# 四 基础操作



基本Rest风格命令说明

| method | url地址                                         | 描述                 |
| ------ | ----------------------------------------------- | -------------------- |
| PUT    | 127.0.0.1:9200/索引名称/类型名称/文档id         | 创建文档（指定id）   |
| POST   | 127.0.0.1:9200/索引名称/类型名称                | 创建文档(随机文档ID) |
| POST   | 127.0.0.1:9200/索引名称/类型名称/文档id/_update | 修改文档             |
| DELETE | 127.0.01:9200/索引名称/类型名称/文档id          | 删除文档             |
| GET    | 127.0.01:9200/索引名称/类型名称/文档id          | 查询文档通过ID       |
| POST   | 127.0.0.1:9200/索引名称/类型名称/_search        | 查询所有数据         |

## 1 基础测试

### 1 创建索引

> PUT /索引名/[文档名]/ID {请求体}

```ruby
PUT /test/type1/1
{
  "name":"jack",
  "age":"18"
}
```

![image-20210717120114392](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210717120116.png)

### 2 添加字段类型

> 类似于mysql建表

[官网教程](https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html)

```ruby
PUT my-index-000001
 
  "mappings": {
    "properties": {
      "name": {
        "type":  "text"
      },
      "age":{
        "type":"long"
      }
    }
  }
}
```

![image-20210717131808699](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210717131809.png)

### 3 获取索引信息

```ruby
GET my-index-000001
```

![image-20210717132024549](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210717132026.png)

> 注意： 如果文档没有指定类型，ES会默认配置字段类型

### 4 获取健康信息

```ruby
GET /_cat/health
```

### 5 获取索引情况

```ruby
GET /_cat/indices?v
```

### 6 修改数据

> PUT会造成覆盖不推荐，POST更灵活

**PUT方式修改  不推荐** 

```ruby
PUT /test/type1/1
{
  "name":"jack",
"age":"18"}
```

![image-20210717203220827](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210717203231.png)

**POST方式修改 推荐**

```ruby
POST /test/type1/1/_update
{
 "doc":{
     "name":"jack123"
 }
}
GET /test/type1/1/
```

![image-20210717203514944](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210717203516.png)

### 7 删除索引

> 根据请求判断删除索引还是删除文档记录
>
> 写文档就直接删除文档，写库就直接删除库

## 2 文档基本操作

### 1 基本操作

> 添加基础数据

```ruby
PUT /cur/user/1
{
  "name":"curiosity",
  "local":"南非",
  "home":["湖南","湖北"]
}
PUT /cur/user/2
{
  "name":"张三",
  "local":"阿拉伯",
  "home":["山东","河南"]
}
PUT /cur/user/3
{
  "name":"李四",
  "local":"熬鹰哥特",
  "home":["海南","重庆"]
}
```

> Version代表更新次数

![image-20210717205106399](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210717205120.png)

#### 1 条件查询

```ruby
GET cur/user/_search?q=name:李四
```

### 2 复杂操作

#### 1 query 查询匹配条件

```ruby
GET cur/user/_search
{
  "query":{
    "match": {
      "name": "李四"
    }
  }
}
```

![image-20210717211040017](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210717201703.png)

**hits:**

* 索引和文档信息
* 查询结果总数
* 查询的具体文档
* max_score可以判断谁是最匹配的结果

#### 2 _source 指定查询出来的字段 

> 类似于mysql的 select 字段

```ruby
GET cur/user/_search
{
  "query":{
    "match": {
      "name": "李四"
    }
  },
  "_source":["name","local"]
}

```

![image-20210717202337149](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210717202338.png)

#### 3 sort 排序

```ruby
GET cur/user/_search
{
  "query":{
    "match": {
      "name": "李四"
    }
  },
  "_source":["name","local"],
  "sort":[
    {
      "age":{
        "order":"asc"
      }
    }
  ]
}
```

#### 4 from size分页

> 数据下标从0开始 

* from 设置从第几个数据开始
* size 返回多少条数据

```ruby
GET cur/user/_search
{
  "query":{
    "match": {
      "name": "李四"
    }
  },
  "_source":["name","local"],
  "from":0,
  "size":2
}

```

#### 5 布尔查询

> 类似于where语句

* must 相当于mysql里面的 and
* should 相当于mysql里面的 or
* must_not 相当于 !=

```ruby
GET cur/user/_search
{
  "query":{
    "bool": {
      "must": [
        {
          "match": {
               "name": "李四"
          },
          "match": {
               "name": "李四"123
          }
        }
      ]
    }
  }
}
```

#### 6 filter 过滤

> 例如：过滤区间

* gt >
* gte >=
* lt <
* lte <=

> 筛选数据

```ruby
GET cur/user/_search
{
  "query":{
    "bool": {
      "must": [
        {
          "match": {
               "name": "李四"
          }
          
        }
  
      ], 
      "filter": {
        "range":{
          "age":{
             "lt": 20
          }
        }
      }
    }
  }
}

```

#### 7 多条件

> 多条件使用空格隔开，只要满足其中一个结果就可以被查出来
>
> 这个时候可以通过分值基本的判断

![image-20210717214317982](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210717214320.png)

#### 8 term 查询

> term查询直接通过倒排索引指定的词条进行精确查找

**分词：**

* term： 直接查询精确的

* match 会使用分词器解析！（先分析文档，然后在通过分析的文档记性查询）



**keyword 类型的不会被分词器解析**

```ruby
PUT testdb
{
  "mappings": {
    "properties": {
      "name":{
        "type": "text"
      },
      "desc":{
        "type": "keyword"
      }
    }
  }
}

PUT testdb/_doc/1
{
  "name":"我的家在东北",
  "desc":"我的家在东北"
}
PUT testdb/_doc/3
{
  "name":"我的家在东北",
  "desc":"我的家在东北3"
}
GET _analyze
{
  "analyzer": "keyword"
  , "text": "我的家在东北"
}

GET _analyze
{
  "analyzer": "standard"
  , "text": "我的家在东北"
}
GET testdb/_search
{
  "query": {
    "term": {
      "desc": "我"
    }
  }
}

```

![image-20210717215353565](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210717220621.png)

#### 9 多个值精确查询

```ruby
GET testdb/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
            "desc": "我的家在东北"
          }
        },
                {
          "term": {
            "desc": "我的家在东北2"
          }
        }
      ]
    }
  }
}
```

#### 10 高亮查询heighligh

```ruby

GET cur/user/_search
{
  "query":{
    "match": {
      "name": "李四"
    }
  },
  "highlight":{
    "fields": {
      "name":{}
    }
  }
}

```

![image-20210717220617879](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210717220627.png)

**自定义高亮**

![image-20210717213257144](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210717213317.png)

# 五 集成SpringBoot

[官方文档](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-getting-started-initialization.html)

**注意：**

> 注意es的版本避免不必要的错误

![image-20210717224133923](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210717224136.png)

## 1 创建索引

```java
@Test
void testCreateIndex() throws IOException {
    // 创建索引请求
    CreateIndexRequest request = new CreateIndexRequest("boot_index");
    // 执行请求
    final CreateIndexResponse createIndexResponse = restHighLevelClient.indices().create(request, RequestOptions.DEFAULT);
    System.out.println(createIndexResponse);
}
```

## 2 判断索引是否存在

```java
@Test
// 获取索引判断是否存在
void getIndex() throws IOException {
    // 获取索引请求
    GetIndexRequest request = new GetIndexRequest("boot_index");
    // 执行请求
    final boolean exists = restHighLevelClient.indices().exists(request, RequestOptions.DEFAULT);
    System.out.println(exists);
}
```

## 3 删除索引

```java
@Test
// 删除索引
void deletreIndex() throws IOException {
    // 获取索引请求
    DeleteIndexRequest request = new DeleteIndexRequest("boot_index");
    // 执行请求
    final AcknowledgedResponse delete = restHighLevelClient.indices().delete(request, RequestOptions.DEFAULT);
    System.out.println(delete.isAcknowledged());
}
```

## 4 创建文档

```java
@Test
//  创建文档
void createDecument() throws IOException {
    User user = new User();
    user.setUser("漩涡鸣人");
    user.setAge(17);
    // 创建请求
    IndexRequest request = new IndexRequest("boot_index");
    // 规则 put /boot_index/_doc/1
    request.id("1");
    request.timeout(TimeValue.timeValueSeconds(1));
    request.timeout("1s");
    ObjectMapper mapper = new ObjectMapper();
    final String string = mapper.writeValueAsString(user);
    request.source(string, XContentType.JSON);
    final IndexResponse index = restHighLevelClient.index(request, RequestOptions.DEFAULT);
    System.out.println(index.status().getStatus());

}
```

## 5 获取文档判断是否存在

```java
@Test
//  获取文档
void getDecument() throws IOException {

    // 创建请求
    GetRequest request = new GetRequest("boot_index", "1");
    // 不获取返回的_source 的上下文
    request.fetchSourceContext(new FetchSourceContext(false));
    request.storedFields("_none_");
    final boolean exists = restHighLevelClient.exists(request, RequestOptions.DEFAULT);
    System.out.println(exists);
}

```

## 6 获取文档

```java
@Test
//  获取文档
void getDecumentSource() throws IOException {

    // 创建请求
    GetRequest request = new GetRequest("boot_index", "1");
    final GetResponse documentFields = restHighLevelClient.get(request, RequestOptions.DEFAULT);
    System.out.println(documentFields.getSourceAsString());
}
```

## 7 更新文档

```java
@Test
//  更新
void update() throws IOException {

    // 创建请求
    UpdateRequest request = new UpdateRequest("boot_index", "1");
    User user = new User();
    user.setUser("纳鲁托");
    user.setAge(17);
    ObjectMapper mapper = new ObjectMapper();
    final String string = mapper.writeValueAsString(user);
    request.doc(string,XContentType.JSON);
    final UpdateResponse update = restHighLevelClient.update(request, RequestOptions.DEFAULT);
    System.out.println(update.getGetResult());
}
```

## 8 删除文档

```java
@Test
//  删除
void delete() throws IOException {

// 创建请求
DeleteRequest request = new DeleteRequest("boot_index", "1");

final DeleteResponse delete = restHighLevelClient.delete(request, RequestOptions.DEFAULT);
System.out.println(delete.status());
}
```

## 9 批量操作

```java
@Test
//  批量导入
void taskBulkRequest() throws IOException {

    // 创建请求
    BulkRequest request = new BulkRequest("boot_index");
    List<User> list = new ArrayList<>();
    for (int i = 0; i < 10; i++) {
        list.add(new User("测试"+i,i));
    }
    ObjectMapper mapper = new ObjectMapper();

    for (int i = 0; i < list.size(); i++) {
        request.add(new IndexRequest("boot_index").id(i+"").
                    source(mapper.writeValueAsString(list.get(i)),XContentType.JSON));

    }
    final BulkResponse bulk = restHighLevelClient.bulk(request, RequestOptions.DEFAULT);
    System.out.println(bulk.status());
}
```

## 10 查询

```java
@Test
//  查询
void query() throws IOException {
    SearchRequest searchRequest = new SearchRequest("boot_index");
    SearchSourceBuilder searchRequestBuilder = new SearchSourceBuilder();
    // 精确
    final TermQueryBuilder termQueryBuilder = QueryBuilders.termQuery("user", "测试1");
    // 匹配所有
    final MatchAllQueryBuilder matchAllQueryBuilder = QueryBuilders.matchAllQuery();
    searchRequestBuilder.query(matchAllQueryBuilder);
    searchRequest.source(searchRequestBuilder);
    final SearchResponse search = restHighLevelClient.search(searchRequest,RequestOptions.DEFAULT);
    final String s = Arrays.stream(search.getHits().getHits()).collect(Collectors.toList()).toString();
    System.out.println(s);
}
```

