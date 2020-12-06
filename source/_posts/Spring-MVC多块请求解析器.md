---
title: Spring MVC多块请求解析器
top: false
cover: false
toc: true
mathjax: true
categories:
  - 源代码 - SpringMvc源码 - 多块请求解析器
tags:
  - 源码 - SpringMvc 多块请求解析器
date: 2020-12-06 18:40:56
password:
summary:
---

# 1 多块请求解析器

> ​		在进入doDispatcher方法后，最先用到的组件是多块请求解析器。多块请求解析器接口 包括以下三个方法。

*  boolean isMultipart(HttpServletRequest request);

  > 判断是否为多块请求

* MultipartHttpServletRequest resolveMultipart(HttpServletRequest reuqest)

  >  用于解析多块请求，返回新的包装类型MultipartHttpServletRequest 封装多块请求相关方法与属性

* void cleanupMultipart(MultipartHttpServletRequest  request);

  > 用于在请求处理完成后对多块请求使用的资源执行清理操作

> 默认的多块请求解析器为StandardServletMultipartResolver。在请求处理时候，先使用isMultipart判断是否为多块请求。

```java
// 判断是否为多块请求
@Override
public boolean isMultipart(HttpServletRequest request) {
//判断前缀
    return StringUtils.startsWithIgnoreCase(request.getContentType(), "multipart/");
}

```

## 1 处理多块请求逻辑

```java
@Override
public MultipartHttpServletRequest resolveMultipart(HttpServletRequest request) throws MultipartException {
    // 对原始request进行包装
    //resolveLazily默认为false,表示实例创建的时候就对请求进行解析
    // 该配置通过spring.servlet.multipart.resolve-lazily修改
   return new StandardMultipartHttpServletRequest(request, this.resolveLazily);
}
```

## 2 **StandardMultipartHttpServletRequest的构造逻辑**

```java
public StandardMultipartHttpServletRequest(HttpServletRequest request, boolean lazyParsing)
      throws MultipartException {
// 调用父类构造器,该类型为对HttpServletRequest的包装类,装饰者模式,用于增强原始类型的功能构造器参数为要包装的对象
   super(request);
    // 如果不是懒加载解析,则直接解析请求
   if (!lazyParsing) {
       //解析请求
      parseRequest(request);
   }
}
```

## 3 **解析请求的方法**

```java
private void parseRequest(HttpServletRequest request) {
   try {
      // 获取多块请求原始的Part类型集合
       // Part结合包括请求参数与请求文件
      
      Collection<Part> parts = request.getParts();
       // 用于保存多块请求的请求参数名称
      this.multipartParameterNames = new LinkedHashSet<>(parts.size());
       // 用于保存多块请求的请求文件
      MultiValueMap<String, MultipartFile> files = new LinkedMultiValueMap<>(parts.size());
      // 遍历多块数据
       for (Part part : parts) {
           //获取当前遍历块的Content-Disposition请求头
         String headerValue = part.getHeader(HttpHeaders.CONTENT_DISPOSITION);
           // 解析Content-Disposition请求头为ContentDisposition类型,其中包括请求头内容类型,文件名等信息
         ContentDisposition disposition = ContentDisposition.parse(headerValue);
           // 获取块请求的文件名
         String filename = disposition.getFilename();
           // 如果文件名不为空
         if (filename != null) {
             // 如果当前块是个文件类型的块,解析文件名
            if (filename.startsWith("=?") && filename.endsWith("?=")) {
               filename = MimeDelegate.decode(filename);
            }
             // 放入文件Map,key是请求参数名称,value是封装文件块请求与文件名的StandardMultipartFile类型
            files.add(part.getName(), new StandardMultipartFile(part, filename));
         }
         else {
             // 不是文件类型的块请求,把块名放入请求参数名集合
            this.multipartParameterNames.add(part.getName());
         }
      }
       // 在本实例保存多块请求文件的map,封装为不可修改的Map,防止处理过程中被篡改
      setMultipartFiles(files);
   }
   catch (Throwable ex) {
       // 发生任何异常交给异常处理逻辑抛出异常
      handleParseFailure(ex);
   }
}
```

## 4 清理资源

```java
@Override
public void cleanupMultipart(MultipartHttpServletRequest request) {
    // 只有是多块请求,且多块请求已经解析的情况下,才执行清理
   if (!(request instanceof AbstractMultipartHttpServletRequest) ||
         ((AbstractMultipartHttpServletRequest) request).isResolved()) {
      // To be on the safe side: explicitly delete the parts,
      // but only actual file parts (for Resin compatibility)
      try {
          // 遍历全部块
         for (Part part : request.getParts()) {
             //如果块中包含文件则进行删除操作,清理占用的临时资源
            if (request.getFile(part.getName()) != null) {
               part.delete();
            }
         }
      }
      catch (Throwable ex) {
         LogFactory.getLog(getClass()).warn("Failed to perform cleanup of multipart items", ex);
      }
   }
}
```