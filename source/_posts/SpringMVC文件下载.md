---
title: SpringMVC文件下载
top: false
cover: false
toc: true
mathjax: true
categories:
  - SpringMVC
  - 基础
tags:
  - SpringMVC
date: 2020-12-06 20:58:51
password:
summary:

---

# SpringMVC 文件下载

# 1 Resource同步文件下载方式

```java
@GetMapping(value = "/test/download/{fileName}")
public ResponseEntity<Resource> downLoad(@PathVariable("fileName") String fileName) throws UnsupportedEncodingException, InterruptedException {
    String filePath = "D:\\00_aa_lijinze_java_web_stady\\" + fileName;
    FileSystemResource resource = new FileSystemResource(filePath);
    MediaType mediaType = MediaTypeFactory
            .getMediaType(resource)
            .orElse(MediaType.APPLICATION_OCTET_STREAM);
    HttpHeaders headers = new HttpHeaders();
    headers.setContentType(mediaType);
    ContentDisposition disposition = ContentDisposition.builder("attachment")
            .filename(new String(resource.getFilename().getBytes("UTF-8"), "ISO-8859-1"))
            .build();
    headers.setContentDisposition(disposition);
    return new ResponseEntity<>(resource, headers, HttpStatus.CREATED);
}


```

# 2 StreamingResponseBody 异步下载方式

### 1 配置Task

```java
@Configuration
@EnableAsync
@EnableScheduling
@Slf4j
public class AsyncConfiguration implements AsyncConfigurer {
    @Override
    @Bean(name = "taskExecutor")
    public AsyncTaskExecutor getAsyncExecutor() {
        log.debug("Creating Async Task Executor");
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(25);
        return executor;
    }

    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return new SimpleAsyncUncaughtExceptionHandler();
    }
    /** Configure async support for Spring MVC. */
    @Bean
    public WebMvcConfigurer webMvcConfigurer(@Qualifier("taskExecutor") AsyncTaskExecutor taskExecutor, CallableProcessingInterceptor callableProcessingInterceptor) {
        return new WebMvcConfigurer() {
            @Override
            public void configureAsyncSupport(AsyncSupportConfigurer configurer) {
                configurer.setDefaultTimeout(360000).setTaskExecutor(taskExecutor);
                configurer.registerCallableInterceptors(callableProcessingInterceptor);
                WebMvcConfigurer.super.configureAsyncSupport(configurer);
            }
        };
    }

    @Bean
    public CallableProcessingInterceptor callableProcessingInterceptor() {
        return new TimeoutCallableProcessingInterceptor() {
            @Override
            public <T> Object handleTimeout(NativeWebRequest request, Callable<T> task) throws Exception {
                log.error("timeout!");
                return super.handleTimeout(request, task);
            }
        };
    }
}

```

### 2 异步下载

```java

@GetMapping(value = "/test/download")
public ResponseEntity<StreamingResponseBody> downLoad2() throws IOExceptionn {
    String filePath = "D:\\00_aa_lijinze_java_web_stady\\《Spring源码深度解析（第2版）》_郝佳- AiBooKs.Cc.pdf";
    FileSystemResource resource = new FileSystemResource(filePath);
    MediaType mediaType = MediaTypeFactory
            .getMediaType(resource)
            .orElse(MediaType.APPLICATION_OCTET_STREAM);
    HttpHeaders headers = new HttpHeaders();s
    headers.setContentType(mediaType);
    ContentDisposition disposition = ContentDisposition.builder("attachment")
            .filename(new String(resource.getFilename().getBytes("UTF-8"), "ISO-8859-1"))
            .build();
    headers.setContentDisposition(disposition);
    return new ResponseEntity<StreamingResponseBody>(outputStream -> {
        try (BufferedOutputStream stream = new BufferedOutputStream(outputStream);
             BufferedInputStream inputStream = new BufferedInputStream(resource.getInputStream());) {
            StreamUtils.copy(inputStream, stream);
    }, headers, HttpStatus.CREATE);

}
```

