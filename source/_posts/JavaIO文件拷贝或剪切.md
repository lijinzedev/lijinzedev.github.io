---
title: JavaIO文件拷贝剪切删除
top: false
cover: false
toc: true
mathjax: true
categories:
  - NIO
tags:
  - 文件拷贝
  - NIO
date: 2021-07-07 22:20:02
password:
summary:
---

# 一 创建写入文件

| 方法名                    |                   |
| ------------------------- | ----------------- |
| Files.newBufferedReader() | Java 8            |
| Files.write()             | Java 7 推荐       |
| PrintWriter               | 一行一行写入      |
| File.createNewFile        |                   |
| FileOutputStream.write    | 管道流,古老但灵活 |

* Files.newBufferedReader()

```java
Path path=Paths.get("D:\\delete\\123\\test.txt");
try (BufferedWriter bufferedWriter = Files.newBufferedWriter(path, StandardCharsets.UTF_8)){
    bufferedWriter.write("hello word");
}
try (BufferedWriter bufferedWriter = Files.newBufferedWriter(path, StandardCharsets.UTF_8,StandardOpenOption.APPEND)){
    bufferedWriter.write("追加");
}
```

* Files.write()

```java
Path path=Paths.get("D:\\delete\\123\\test.txt");
Files.write(path,"hello world".getBytes(StandardCharsets.UTF_8));
Files.write(path,"hello world".getBytes(StandardCharsets.UTF_8),StandardOpenOption.APPEND);
```

* PrintWriter

```java
final String fileName = "D:\\delete\\123\\test.txt";
try (PrintWriter writer=new PrintWriter(fileName,"UTF-8")){
    writer.println("hello world");
}
```

* File.createNewFile

```java
final String fileName = "D:\\delete\\123\\test.txt";
File file=new File(fileName);
file.createNewFile();
try (FileWriter writer=new FileWriter(file)){ 
    writer.write("hello");
}
```

* FileOutputStream.write

```java
final String fileName = "D:\\delete\\123\\test.txt";
try (FileOutputStream fos =new FileOutputStream(fileName);
     OutputStreamWriter ios =new OutputStreamWriter(fos);
     BufferedWriter writer=new BufferedWriter(ios)
    ){
    writer.write("hello");
    writer.flush();
}
```

# 二 从文件中读取数据

- `Scanner`(Java 1.5) 按行读数据及String、Int类型等按分隔符读数据。
- `Files.lines`, 返回`Stream` 流式数据处理，按行读取
- `Files.readAllLines`, 返回`List<String>`
- `Files.readString`, 读取`String`(Java 11), 文件最大 2G.
- `Files.readAllBytes`, 读取`byte[]`(Java 7), 文件最大 2G.
- `BufferedReader`, 经典方式 (Java 1.1 -> forever)

## 1.Scanner

> 第一种方式是Scanner，从JDK1.5开始提供的API，特点是可以按行读取、按分割符去读取文件数据，既可以读取String类型，也可以读取Int类型、Long类型等基础数据类型的数据。

```java
@Test
void testReadFile1() throws IOException {
   //文件内容：Hello World|Hello Zimug
   String fileName = "D:\data\test\newFile4.txt";

   try (Scanner sc = new Scanner(new FileReader(fileName))) {
      while (sc.hasNextLine()) {  //按行读取字符串
         String line = sc.nextLine();
         System.out.println(line);
      }
   }

   try (Scanner sc = new Scanner(new FileReader(fileName))) {
      sc.useDelimiter("\|");  //分隔符
      while (sc.hasNext()) {   //按分隔符读取字符串
         String str = sc.next();
         System.out.println(str);
      }
   }

   //sc.hasNextInt() 、hasNextFloat() 、基础数据类型等等等等。
   //文件内容：1|2
   fileName = "D:\data\test\newFile5.txt";
   try (Scanner sc = new Scanner(new FileReader(fileName))) {
      sc.useDelimiter("\|");  //分隔符
      while (sc.hasNextInt()) {   //按分隔符读取Int
          int intValue = sc.nextInt();
         System.out.println(intValue);
      }
   }
}
```

## 2.Files.lines 

> 按行处理数据文件的内容

```java
@Test
void testReadFile2() throws IOException {
   String fileName = "D:\data\test\newFile.txt";

   // 读取文件内容到Stream流中，按行读取
   Stream<String> lines = Files.lines(Paths.get(fileName));

   // 随机行顺序进行数据处理
   lines.forEach(ele -> {
      System.out.println(ele);
   });
}
```

> forEach获取Stream流中的行数据不能保证顺序，但速度快。如果你想按顺序去处理文件中的行数据，可以使用forEachOrdered，但处理效率会下降。

```java
// 按文件行顺序进行处理
lines.forEachOrdered(System.out::println);
```

或者利用CPU多和的能力，进行数据的并行处理parallel()，适合比较大的文件。

```java
// 按文件行顺序进行处理
lines.parallel().forEachOrdered(System.out::println);
```

也可以把`Stream<String>`转换成`List<String>`,但是要注意这意味着你要将所有的数据一次性加载到内存，要注意[java](http://www.zimug.com/tag/java).lang.OutOfMemoryError

```java
// 转换成List<String>, 要注意java.lang.OutOfMemoryError: Java heap space
List<String> collect = lines.collect(Collectors.toList());
```

## 3.Files.readAllLines

这种方法仍然是java8 为我们提供的，如果我们不需要`Stream<String>`,我们想直接按行读取文件获取到一个`List<String>`，就采用下面的方法。同样的问题：这意味着你要将所有的数据一次性加载到内存，要注意java.lang.OutOfMemoryError

```
@Test
void testReadFile3() throws IOException {
   String fileName = "D:\data\test\newFile3.txt";

   // 转换成List<String>, 要注意java.lang.OutOfMemoryError: Java heap space
   List<String> lines = Files.readAllLines(Paths.get(fileName),
               StandardCharsets.UTF_8);
   lines.forEach(System.out::println);

}
```

## 4.Files.readString(JDK 11)

从 java11开始，为我们提供了一次性读取一个文件的方法。文件不能超过2G，同时要注意你的服务器及JVM内存。**这种方法适合快速读取小文本文件。**

```
@Test
void testReadFile4() throws IOException {
   String fileName = "D:\data\test\newFile3.txt";

   // java 11 开始提供的方法，读取文件不能超过2G，与你的内存息息相关
   //String s = Files.readString(Paths.get(fileName));
}
```

## 5.Files.readAllBytes()

如果你没有JDK11（readAllBytes()始于JDK7）,仍然想一次性的快速读取一个文件的内容转为String，该怎么办？先将数据读取为二进制数组，然后转换成String内容。**这种方法适合在没有JDK11的请开给你下，快速读取小文本文件。**

```
@Test
void testReadFile5() throws IOException {
   String fileName = "D:\data\test\newFile3.txt";

   //如果是JDK11用上面的方法，如果不是用这个方法也很容易
   byte[] bytes = Files.readAllBytes(Paths.get(fileName));

   String content = new String(bytes, StandardCharsets.UTF_8);
   System.out.println(content);
}
```

## 6.经典管道流的方式

最后一种就是经典的管道流的方式

```
@Test
void testReadFile6() throws IOException {
   String fileName = "D:\data\test\newFile3.txt";

   // 带缓冲的流读取，默认缓冲区8k
   try (BufferedReader br = new BufferedReader(new FileReader(fileName))){
      String line;
      while ((line = br.readLine()) != null) {
         System.out.println(line);
      }
   }

   //java 8中这样写也可以
   try (BufferedReader br = Files.newBufferedReader(Paths.get(fileName))){
      String line;
      while ((line = br.readLine()) != null) {
         System.out.println(line);
      }
   }

}
```

这种方式可以通过管道流嵌套的方式，组合使用，比较灵活。比如我们
想从文件中读取java Object就可以使用下面的代码，前提是文件中的数据是ObjectOutputStream写入的数据，才可以用ObjectInputStream来读取。

```
try (FileInputStream fis = new FileInputStream(fileName);
     ObjectInputStream ois = new ObjectInputStream(fis)){
   ois.readObject();
} 
```

# 

# 三 文件复制,剪切,重命名

## 1.传统文件复制

> 缺点:文件强行覆盖对使用者无感知

```java
File file1 = new File("/home/a.text");
File file2 = new File("/home/b.text");
try (FileInputStream in = new FileInputStream(file1)
     ; FileOutputStream out = new FileOutputStream(file2)) {
    byte[] buffer = new byte[1024];
    int length;
    while ((length=in.read(buffer))>0){
        out.write(buffer,0,length);
        out.flush();
    }
}
```

## 2.NIO文件复制与重命名(推荐)

```java
Path formFile = Paths.get("/home/a.text");
Path toFile = Paths.get("/home/b.text");
// 源文件不存在会抛出异常,目标文件不存在也会抛出异常
Files.copy(formFile, toFile);
// 如果目标文件已经存在就直接覆盖
Files.copy(formFile, toFile, StandardCopyOption.REPLACE_EXISTING);

// 连文件属性也一起拷贝
CopyOption[] options = {StandardCopyOption.REPLACE_EXISTING, StandardCopyOption.COPY_ATTRIBUTES};
Files.copy(formFile, toFile, options);

// 文件重命名
Files.move(formFile, toFile, StandardCopyOption.REPLACE_EXISTING);

// 更好的文件重命名
// 兼容性更好
Files.move(formFile, formFile.resolveSibling("b.text"));
```

## 3. NIO文件剪切

```java
Path formFile = Paths.get("/home/a.text");
Path anotherDir = Paths.get("/home/user/");
// 文件剪切
Files.createDirectories(anotherDir);
Files.move(formFile, anotherDir.resolve(formFile.getFileName()), StandardCopyOption.REPLACE_EXISTING);
```

# 四 文件删除,文件夹删除

> NIO使用Path 代表文件或者文件夹
>
> 传统IO使用File 代表文件或者文件夹

## 1. 删除文件或者空文件夹

|                               | 成功返回值 | 是否能判断文件夹不存在导致失败 | 是否能判断文件夹位空导致失败 | 备注        |
| ----------------------------- | ---------- | ------------------------------ | ---------------------------- | ----------- |
| File.delete()                 | true       | 不能(返回false)                | 不能(返回false)              | 传统不推荐  |
| File.deleteOnExit()           | void       | 不能 但存在就不会执行删除      | 不能返回void                 | 不适用,有坑 |
| Files.delete(Path path)       | void       | NoSuchFileException            | DirectoryNotEmptyException   | NIO推荐     |
| Files.deleteExists(Path path) | true       | false                          | DirectoryNotEmptyException   | NIO         |

### 1.传统IO

```java
// 文件删除
File file1 = new File("/home/a.text");
boolean delete = file1.delete();
```

### 2. NIO

```java
// 文件删除,可以抛出异常
Path formFile = Paths.get("/home/a.text");
Files.delete(formFile);
```

## 2. 删除非空文件夹或者某一些符合条件的文件

* walkFileTree与FileVisitor

> 可以通过BasicFileAttributes进行条件删除

```java
String pathname = "D:\\delete\\123";
Path path = Paths.get(pathname);
Files.walkFileTree(path, new SimpleFileVisitor<Path>() {
    // 先遍历删除文件
    @Override
    public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
        Files.delete(file);
        System.out.printf("删除文件 :$s:n",file);
        return FileVisitResult.CONTINUE;
    }

    // 后遍历删除目录
    @Override
    public FileVisitResult postVisitDirectory(Path dir, IOException exc) throws IOException {
        // 此时文件已经被删除了,这时候文件夹是空的
        Files.delete(dir);
        System.out.printf("删除文件 :$s:n",dir);
        return FileVisitResult.CONTINUE;
    }
});
```

* Files.walk

> 排序的目的是文件排在目录前面,先去删除文件,再去删除目录

```java
String pathname = "D:\\delete\\123";
Path path = Paths.get(pathname);
try (Stream<Path> walk = Files.walk(path)){
    walk.sorted(Comparator.reverseOrder()).forEach(path1 -> {
        try {
            Files.delete(path1);
        } catch (IOException e) {
            e.printStackTrace();
        }
    });
}
```

* 递归listFiles

```java
String pathname = "D:\\delete\\123";
delete(new File(pathname));

public static void delete(File file) {
    File[] files = file.listFiles();
    if (files != null) {
        Arrays.stream(files).forEach(file1 -> delete(file));
    }
    file.delete();
}
```

# 五  创建文件夹

## 1.传统API创建文件夹方式

Java传统的IO API种使用`java.io.File`类中的`file.mkdir()`和`file.mkdirs()`方法创建文件夹

- `file.mkdir()`创建文件夹成功返回true，失败返回false。如果被创建文件夹的父文件夹不存在也返回false.没有异常抛出。
- `file.mkdirs()`创建文件夹连同该文件夹的父文件夹，如果创建成功返回true，创建失败返回false。创建失败同样没有异常抛出。

```java
@Test
void testCreateDir1() {
   //“D:\data111”目录现在不存在
   String dirStr = "D:\data111\test";
   File directory = new File(dirStr);

   //mkdir
   boolean hasSucceeded = directory.mkdir();
   System.out.println("创建文件夹结果（不含父文件夹）：" + hasSucceeded);

   //mkdirs
   hasSucceeded = directory.mkdirs();
   System.out.println("创建文件夹结果（包含父文件夹）：" + hasSucceeded);

}
```

输出结果如下：使用mkdir创建失败，使用mkdirs创建成功。

```java
创建文件夹结果（不含父文件夹）：false
创建文件夹结果（包含父文件夹）：true
```

大家可以看到，mkdir和mkdirs虽然可以创建文件，但是它们在异常处理的环节做的非常不友好。创建失败之后统一返回false，创建失败的原因没有说明。是父文件夹不存在所以创建失败？还是文件夹已经存在所以创建失败？还是因为磁盘IO原因导致创建文件夹失败？

## 2. Java NIO创建文件夹

为了解决传统IO创建文件夹中异常失败处理问题不明确的问题，在Java的NIO中进行了改进。

### 2.1. `Files.createDirectory`创建文件夹

- 如果被创建文件夹的父文件夹不存在，则抛出`NoSuchFileException`.
- 如果被创建的文件夹已经存在，则抛出`FileAlreadyExistsException`.
- 如果因为磁盘IO出现异常，则抛出`IOException`.

```java
Path path = Paths.get("D:\data222\test");
Path pathCreate = Files.createDirectory(path);
```

### 2.2.`Files.createDirectories`创建文件夹及其父文件夹

- 如果被创建文件夹的父文件夹不存在，就创建它
- 如果被创建的文件夹已经存在，就是用已经存在的文件夹，不会重复创建，没有异常抛出
- 如果因为磁盘IO出现异常，则抛出`IOException`.

```java
Path path = Paths.get("D:\data222\test");
Path pathCreate = Files.createDirectorys(path);
```

注意：NIO的API创建的文件夹返回值是Path，这样方便我们在创建完成文件夹之后继续向文件夹里面写入文件数据等操作。比传统IO只返回一个boolean值要好得多。
