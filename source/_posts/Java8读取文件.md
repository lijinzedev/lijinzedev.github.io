---
title: Java8读取文件
top: false
cover: false
toc: true
mathjax: true
categories:
  - Java - Stream
tags:
  - Java - 文件读取
date: 2021-05-19 13:52:47
password:
summary:
---

# 一、Jdk 8 流

## 1 Stream + Files.lines

```java
String fileName = "C:\\Users\\king\\Desktop\\nginx.conf";
Stream<String> lines = Files.lines(Paths.get(fileName));
System.out.println();
lines.forEach(System.out::println);
```

## 2 Stream + BufferedReader 

```java
String fileName = "C:\\Users\\king\\Desktop\\nginx.conf";
BufferedReader bufferedReader = Files.newBufferedReader(Paths.get(fileName));
Stream<String> lines = bufferedReader.lines();
lines.forEach(System.out::println);
```

# 二、JDK1.1 BufferedReader 

```java
String fileName = "C:\\Users\\king\\Desktop\\nginx.conf";
try (BufferedReader br = new BufferedReader(new FileReader(fileName))) {

    String line;
    while ((line = br.readLine()) != null) {
        System.out.println(line);
    }

} catch (IOException e) {
    e.printStackTrace();
}
```

# 三、 JDK1.5 Scanner

```java
String fileName = "C:\\Users\\king\\Desktop\\nginx.conf";
try (Scanner scanner = new Scanner(new File(fileName))) {

    while (scanner.hasNext()){
        System.out.println(scanner.nextLine());
    }

} catch (IOException e) {
    e.printStackTrace();
}
```