---
title: SpringMVC文件下载
top: false
cover: false
toc: true
mathjax: true
categories:
  - SpringMVC - 功能
tags:
  - SpringMVC - 文件下载
date: 2020-12-06 20:58:51
password:
summary:
---

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


@GetMapping(value = "/test/download")
public ResponseEntity<StreamingResponseBody> downLoad2() throws UnsupportedEncodingException {
    String filePath = "D:\\00_aa_lijinze_java_web_stady\\《Spring源码深度解析（第2版）》_郝佳- AiBooKs.Cc.pdf";
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
    return new ResponseEntity<StreamingResponseBody>(outputStream -> {
        int bytesRead;
        byte[] buffer = new byte[BUFFER_SIZE];
        InputStream inputStream = resource.getInputStream();
        while ((bytesRead = inputStream.read(buffer)) != -1) {
            outputStream.write(buffer, 0, bytesRead);
        }
    }, headers, HttpStatus.CREATED);
}
```