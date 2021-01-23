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



| 名称                | 解释                                                         |
| ------------------- | ------------------------------------------------------------ |
| @RequestLine        | 方法上 定义HttpMethod 和 UriTemplate. UriTemplate 中使用{} 包裹的表达式，可以通过在方法参数上使用 |
| @Param              | 自动注入 @Param 方法参数 定义模板变量，模板变量的值可以使用名称的方式使用模板注入解析 |
| @Headers            | 类上或者方法上 定义头部模板变量，使用@Param 注解提供参数值的注入。如果该注解添加在接口类上，则所有的请求都会携带对应的Header信息；如果在方法上，则只会添加到对应的方法请求上 |
| @QueryMap           | 方法上 定义一个键值对或者 pojo，参数值将会被转换成URL上的 query 字符串上 |
| @HeaderMap          | 方法上 定义一个HeaderMap, 与 UrlTemplate 和HeaderTemplate 类型，可以使用@Param 注解提供参数值 |
| @FeignClient        | 注解指定调用的微服务名称，封装了调用 USER-API 的过程，作为消费方调用模板。 |
| @EnableFeignClients | 扫描声明它们是假装客户端的接口（@FeignClient ）。配置组件扫描指令以供使用(@Configuration )类。 |
| @SpringQueryMap     | Spring MVC相当于OpenFeign的{@QueryMap} 。                    |

## @FeignClient详解

| 名称                   | 解释                                                         |
| ---------------------- | ------------------------------------------------------------ |
| name、value和serviceId | 从源码可以得知，name是value的别名，value也是name的别名。两者的作用是一致的，name指定FeignClient的名称，如果项目使用了Ribbon，name属性会作为微服务的名称，用于服务发现。其中，serviceId和value的作用一样，用于指定服务ID，已经废弃。 |
| qualifier              | 该属性用来指定@Qualifier注解的值，该值是该FeignClient的限定词，可以使用改值进行引用。 |
| url                    | url属性一般用于调试程序，允许我们手动指定@FeignClient调用的地址。 |
| decode404              | 当发生http 404错误时，如果该字段位true，会调用decoder进行解码，否则抛出FeignException |
| configuration          | Feign配置类，可以自定义Feign的Encoder、Decoder、LogLevel、Contract。 |
| fallback               | 定义容错的处理类，当调用远程接口失败或超时时，会调用对应接口的容错逻辑，fallback指定的类必须实现@FeignClient标记的接口 |
| fallbackFactory        | 工厂类，用于生成fallback类示例，通过这个属性我们可以实现每个接口通用的容错逻辑，减少重复的代码。 |
| path                   | path属性定义当前FeignClient的统一前缀。                      |
| primary                | 是否将伪代理标记为主Bean，默认为true。                       |

## @EnableFeignClients

| 名称                 | 解释                                                         |
| -------------------- | ------------------------------------------------------------ |
| value                | 为basePackages属性的别名，允许使用更简洁的书写方式。         |
| basePackage          | 设置自动扫描带有@FeignClient注解的基础包路径。               |
| basePackageClasses   | 该属性是basePackages属性的安全替代属性。该属性将扫描指定的每个类所在的包下面的所有被@FeignClient修饰的类；这需要考虑在每个包中创建一个特殊的标记类或接口，该类或接口除了被该属性引用外，没有其他用途。 |
| defaultConfiguration | 该属性用来自定义所有Feign客户端的配置，使用@Configuration进行配置。 |
| clients              | 设置由@FeignClient注解修饰的类列表。如果clients不是空数组，则不通过类路径自动扫描功能来加载FeignClient。 |



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



> 默认情况下，在SpringCloud Alibaba 微服务项目中，无法通过消费者服务直接将前端上传的文件，以请求的方式发送到服务提供者来操作文件上传的。
>
> 因此需要在 OpenFeign 中将 文件名称 和 文件内容的Base64 封装为类，消费者添加 consumes 文件上传参数。

### 2. 文件上传

#### 2.1 配置类

```java
public  class CoreFeignConfiguration {
    @Bean
    public Request.Options options() {
        return new Request.Options(6 * 1000, 6 * 1000);
    }
    @Autowired
    private ObjectFactory<HttpMessageConverters> messageConverters;

    @Bean
    public Encoder multipartFormEncoder() {
        return new SpringFormEncoder(new SpringEncoder(messageConverters));
    }

    @Bean
    public feign.Logger.Level multipartLoggerLevel() {
        return feign.Logger.Level.FULL;
    }
}
```

#### 2.2 接口

> 消费者 OssService 接口(`@FeignClient`注解修饰不需要实现类)，
>
> 添加参数 `consumes = MediaType.MULTIPART_FORM_DATA_VALUE`

```java
/**
 * @author: Curiosity
 * @Date: 2021/1/23 17:46
 * @Description:
 */
@FeignClient(name = "nacos-config-client", path = "/feign", configuration = {TestFeign2.CoreFeignConfiguration.class})
public interface TestFeign2 {
    @PostMapping("/get")
    public String providerFeign(@RequestBody Payment payment);

    @PostMapping(value = "/upload",consumes = MediaType.MULTIPART_FORM_DATA_VALUE,produces = MediaType.APPLICATION_JSON_VALUE)
    String upload(@RequestPart("file") MultipartFile multipartFile);
	// 局部配置
    public static class CoreFeignConfiguration {
        @Bean
        public Request.Options options() {
            return new Request.Options(6 * 1000, 6 * 1000);
        }
        @Autowired
        private ObjectFactory<HttpMessageConverters> messageConverters;

        @Bean
        public Encoder multipartFormEncoder() {
            return new SpringFormEncoder(new SpringEncoder(messageConverters));
        }

        @Bean
        public feign.Logger.Level multipartLoggerLevel() {
            return feign.Logger.Level.FULL;
        }
    }
}
```

#### 2.3 文件转化为MultipartFile

```java 
// 方法 1  
File file = new File("src/test/resources/input.txt");
    FileInputStream input = new FileInputStream(file);
    MultipartFile multipartFile = new MockMultipartFile("file",
            file.getName(), "text/plain", IOUtils.toByteArray(input));
// 方法 2
public static MultipartFile file2MultipartFile(File file) throws IOException {
        FileItem fileItem = new DiskFileItemFactory().createItem("file",
                Files.probeContentType(file.toPath()), false, file.getName());
        try (InputStream in = new FileInputStream(file); OutputStream out = fileItem.getOutputStream()) {
            StreamUtils.copy(in,out);
        } catch (Exception e) {
            throw new IllegalArgumentException("Invalid file: " + e, e);
        }

        return new CommonsMultipartFile(fileItem);
    }

```

#### 2.4 实例2 多文件上传

**方法一** (`测试无效,实际上只会穿一个文件`)

```java
@PostMapping(value = "/upload",consumes = MediaType.MULTIPART_FORM_DATA_VALUE,produces = MediaType.APPLICATION_JSON_VALUE)
String upload(@RequestPart("file") MultipartFile[] multipartFile);
```
##### 方法二

`删除Encoder配置`

```java
@Bean
public Encoder multipartFormEncoder() {
    return new SpringFormEncoder(new SpringEncoder(messageConverters));
}
```
**ProviderController**

```java
@PostMapping(path = "/add", consumes = MULTIPART_FORM_DATA_VALUE, produces = APPLICATION_JSON_VALUE)
 public MyResponseObject add(@RequestParam(name = "username") String username,
                             @RequestPart(name = "filetoupload") MultipartFile file) {
              Do something
}
```

**OpenFeign接口**

```java
@PostMapping(path = "/myApi/add", consumes = MULTIPART_FORM_DATA_VALUE, 
              produces = APPLICATION_JSON_VALUE)
 public MyResponseObject addFile(@RequestParam(name = "username") String username,
                           @RequestPart(name = "filetoupload") MultiValueMap<String, Object> file);
```

**转化**

```java
public MyResponseObject addFileInAnotherEndPoint(String username, MultipartFile file) throws IOException {

    MultiValueMap<String, Object> multiValueMap = new LinkedMultiValueMap<>();
    ByteArrayResource contentsAsResource = new ByteArrayResource(file.getBytes()) {
        @Override
        public String getFilename() {
            return file.getOriginalFilename();
        }
    };
    multiValueMap.add("filetoupload", contentsAsResource);
    multiValueMap.add("fileType", file.getContentType());
    return this.myFeignClient.addFile(username, multiValueMap);
}
```

### 3 文件下载

```java
import feign.Response;

@FeignClient(value = "some-service")
public interface Client{
   @RequestMapping(method = RequestMethod.GET, value ="/download")
   Response downloadFile();
}
```

Usage of Feign Client:

```java
final Response response = client.downloadFile();
final Response.Body body = response.body();
final InputStream inputStream = body.asInputStream();
```

## 3 超时时间

> 配置OpenFeign的超时控制时间，OpenFeign 内与 ribbon 整合了，支持负载均衡，它的超时控制也由最底层的 ribbon 进行控制，yml 添加配置

```yaml
 feign:
  client:
    config:
      default:
        connectTimeout: 10000 #设置连接的超时时间 10s
        readTimeout: 20000  #设置读取的超时时间 20s
```

### 局部超时

```java
/**
 * @author: Curiosity
 * @Date: 2021/1/23 17:46
 * @Description:
 */
@FeignClient(name = "nacos-config-client", path = "/feign", configuration = {TestFeign2.CoreFeignConfiguration.class})
public interface TestFeign2 {
    @PostMapping("/get")
    public String providerFeign(@RequestBody Payment payment);

    public static class CoreFeignConfiguration {
        @Bean
        public Request.Options options() {
            return new Request.Options(6 * 1000, 6 * 1000);
        }
    }
}

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

