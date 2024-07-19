---
title: logback1.2.3版本 日志超过999不会自动删除，导致磁盘堆积问题
top: false
cover: false
toc: true
mathjax: true
categories:
  - 源码
  - springboot
tags:
  - logback
  - 踩坑
date: 2024-03-27 14:33:52
password:
summary:
---

# 环境参数说明

- `Springboot 2.5.2`
- `JDK 1.8`
- `logback 1.2.3`

# 一、问题现象

生产环境产生日志堆积，debug日志占用1TB，本地测试日志index在增长到999后，`maxFileSize`、`maxHistory`、`totalSizeCap`均失效，日志无法删除。

# 二、前置知识

https://www.cnblogs.com/cainiao-Shun666/p/16348448.html

## 2.1.基于时间的滚动策略

为了配置滚动日志，我们可以使用`TimeBasedRollingPolicy` ，它具有以下属性。

- `fileNamePattern` ：定义了滚动（归档）日志文件的名称。滚动期是由其值中指定的日期模式推断出来的。默认模式为'yyyy-MM-dd'。
- `maxHistory` （可选）：控制要保留的最大归档文件数量，异步删除旧文件。
- `totalSizeCap` （可选）：控制所有存档文件的总大小。当超过总大小上限时，最老的档案会被异步删除。
- `cleanHistoryOnStart` ：默认情况下是*假的*，这意味着档案移除通常是在滚动期间进行。如果设置为 "true"，档案删除将在appender启动时执行。

谨慎模式支持多个JVM写到同一个日志文件。

给定的日志回溯配置**每天**创建**滚动的日志，保留30天的历史，但最多有3GB的总归档日志**。一旦突破了大小限制，较旧的日志就开始被删除。

```xml
<configuration>
  <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>application.log</file>
    <prudent>true</prudent>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <fileNamePattern>application.%d{yyyy-MM-dd}.log</fileNamePattern>
      <maxHistory>30</maxHistory>
      <totalSizeCap>3GB</totalSizeCap>
    </rollingPolicy>

    <encoder>
      <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
    </encoder>
  </appender> 

  <root level="DEBUG">
    <appender-ref ref="FILE" />
  </root>
</configuration>

```

## 2.2.基于大小和时间的滚动策略

基于大小的滚动策略允许对每个日志文件进行基于文件的滚动。例如，当日志文件的大小达到10MB时，我们可以滚动到一个新的文件。

`maxFileSize` 是用来指定每个文件被滚动时的大小。

另外，注意到令牌`'%i'` ，它被用来创建具有递增索引的新文件，从0开始。这需要创建多个具有`maxFileSize` 限制的小文件，以取代单一的大滚存文件。

给定的logback配置**每天**创建**最大日志大小为10MB的滚动日志，保留30天的历史，但最多有10GB的总存档日志**。一旦超过大小限制，较旧的日志就开始被删除

```xml
<configuration>
  <appender name="ROLLING" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>application.log</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
      <fileNamePattern>application-%d{yyyy-MM-dd}.%i.log.gz</fileNamePattern>
       <maxFileSize>10MB</maxFileSize>    
       <maxHistory>30</maxHistory>
       <totalSizeCap>10GB</totalSizeCap>
    </rollingPolicy>
    <encoder>
      <pattern>%msg%n</pattern>
    </encoder>
  </appender>


  <root level="DEBUG">
    <appender-ref ref="ROLLING" />
  </root>
</configuration>

```

## 2.3.基于大小的触发策略

我们已经在上一节讨论了基于大小的触发策略，使用`maxFileSize` 。它也是如此，只是在语法上有一点区别，`SizeBasedTriggeringPolicy` ，在一个单独的章节中声明。

```xml
<rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
  <fileNamePattern>application-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
  <maxHistory>60</maxHistory>
  <totalSizeCap>20GB</totalSizeCap>
</rollingPolicy>

<triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
  <maxFileSize>10MB</maxFileSize>
</triggeringPolicy>

```

# 三、问题排查

`logback`日志策略如下所示：

1. 日志文件大小是`10MB`
2. 日志文件保留`7`天
3. `info`目录下的所有日志文件总磁盘占用量设置`1GB`

```xml
    <appender name="INFO_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文件的路径及文件名 -->
        <file>${log.path}/log_info.log</file>
        <!--日志文件输出格式-->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 每天日志归档路径以及格式 -->
            <fileNamePattern>${log.path}/info/log-info-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <totalSizeCap>1GB</totalSizeCap>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>10MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文件保留天数-->
            <maxHistory>7</maxHistory>
        </rollingPolicy>
        <!-- 此日志文件只记录info级别的 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>info</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

```

面向Google查询相关问题，看到一个logback无法处理totalSizeCap 超过2GB的bug

```java
void capTotalSize(Date now) {
    int totalSize = 0;
    int totalRemoved = 0;
    for (int offset = 0; offset < maxHistory; offset++) {
        Date date = rc.getEndOfNextNthPeriod(now, -offset);
        File[] matchingFileArray = getFilesInPeriod(date);
        descendingSortByLastModified(matchingFileArray);
        for (File f : matchingFileArray) {
            long size = f.length();
            if (totalSize + size > totalSizeCap) {
                addInfo("Deleting [" + f + "]" + " of size " + new FileSize(size));
                totalRemoved += size;
                f.delete();
            }
            totalSize += size;
        }
    }
    addInfo("Removed  " + new FileSize(totalRemoved) + " of files");
}
```

这段代码定义了totalSize来累加当前日志文件大小，但是数据类型是int，我们知道int类型最大值是 2147483647，而如果totalSize超过了2GB `totalSize + size > totalSizeCap`会不成立。。。。

所以一旦totalSizeCap超过2GB，那么就会导致日志清除失效。

这个问题有人反馈过 [issue](https://jira.qos.ch/browse/LOGBACK-1231)，而logback也在1.2.0版本修复了这个问题，[解决方案是修改int为long类型](https://github.com/qos-ch/logback/commit/2240d4253c9f3fa28497e74f34be3bb1edc73078#diff-cb428af44f615094fb61e87a9ca3411a)。

![commit_2240d42](commit_2240d42.png)

但是这个并不适用于线上遇到的情况，因为服务中使用的是1.2.3版本，于是继续查询分析。又发现了一个[类似报告](https://jira.qos.ch/browse/LOGBACK-1500)，而且报告的版本也是1.2.3

> I’m using SizeAndTimeBasedRollingPolicy.
> When ‘%i’ file index reaches 999 it stops deleting the old files and totalSizeCap is not respected any more.
> This soon leads to disk full issues (as logging in my case was fast enough)

这个问题有点相似了，看一下生产环境的日志文件，前几天都是只保留了后缀1000以上的，当天的保留了700以上的，也就是删除了一部分，后面未删除，而且感觉和这个999又很大关系，这个issue里面没有相关回复，那我们自己一边查看源码一般继续Google吧，上面的源码我们已经看到是日志大小回滚的实现，`File[] matchingFileArray = getFilesInPeriod(date)`，这一步应该是获取相关的文件数组，我们进去继续查看

```java
protected File[] getFilesInPeriod(Date dateOfPeriodToClean) {
        File archive0 = new File(fileNamePattern.convertMultipleArguments(dateOfPeriodToClean, 0));
        File parentDir = getParentDir(archive0);
        String stemRegex = createStemRegex(dateOfPeriodToClean);
        File[] matchingFileArray = FileFilterUtil.filesInFolderMatchingStemRegex(parentDir, stemRegex);
        return matchingFileArray;
    }
 private String createStemRegex(final Date dateOfPeriodToClean) {
        String regex = fileNamePattern.toRegexForFixedDate(dateOfPeriodToClean);
        return FileFilterUtil.afterLastSlash(regex);
    }
```

上面这一块的逻辑是生成一个正则去和日志目录下的日志匹配获取日志文件，下面的代码就是具体拼接正则的实现

复制

```java
/**
     * Given date, convert this instance to a regular expression.
     *
     * Used to compute sub-regex when the pattern has both %d and %i, and the
     * date is known.
     * 
     * @param date - known date
     */
    public String toRegexForFixedDate(Date date) {
        StringBuilder buf = new StringBuilder();
        Converter<Object> p = headTokenConverter;
        while (p != null) {
            if (p instanceof LiteralConverter) {
                buf.append(p.convert(null));
            } else if (p instanceof IntegerTokenConverter) {
                buf.append("(\\d{1,3})");
            } else if (p instanceof DateTokenConverter) {
                buf.append(p.convert(date));
            }
            p = p.getNext();
        }
        return buf.toString();
    }
```

看到这一块果然发现一个问题`buf.append("(\\d{1,3})")`，这个正则是匹配1位到3位的数字，这不刚好，只能识别日志文件后缀1-999，999以上就无法匹配了。

同样我们也找到了相关的[issue](https://jira.qos.ch/browse/LOGBACK-1297)

> I found cause.
>
> Check the **toRegexForFixedDate()** method in **ch.qos.logback.core.rolling.helper.FileNamePattern.java**
>
> Regular expression hardcoded like this:
>
> ***buf.append(“(\ \d{1,3})”);\***
>
> So, files indexed more than 3-digit number are not visible to delete…
>
> I don’t know why the expression hardcoded.
>
> Anyway, you’d better modify the source.

有人提到了现在已经修复

> this issue fixed in 1.3.0-alpha1
>
> https://jira.qos.ch/browse/LOGBACK-1175

果然这里存在问题，[1.3.0-alpha1版本修复这个](https://github.com/qos-ch/logback/commit/f264607fb450)

![commit_f264607]commit_f264607.png)

但是为什么出现了前几天日志文件保留了后缀999以上，当天存在部分低于999的呢？其实这个也很好理解，还是上面源码，先正则匹配到文件之后循环累加大小，maxFileSize =20MB,totalSizeCap =5GB= 5120MB，等于256个文件，256不能被999整除，还会剩余231，也就是说第一天清理了768个文件后，剩余 231 * 20MB<5120MB ,所以不会清理掉，第二天才会继续累加这部分，然后删除上一天剩余的后缀小于1000的日志文件，这样就导致每天可能剩余一定量的后缀小于1000的日志文件，直到第二天被清理，这和我们的实际情况是相符的。

顺便吐槽一下，从GitHub的文件history来看，2012年作者已经改过一次这个正则了，把[d{1,2}改成 d{1,3}](https://github.com/qos-ch/logback/commit/608ed4c58717296b8182006e35aff52ca2ecb598#diff-80c6c7b6b5e9e46b2e5afd6ec7eb07fd) 

![commit_608ed4c](commit_608ed4c.png)

# 四、问题解决

问题已经找出，那么解决就很简单，升级logback版本就行，当然我觉得首先这个日志打印问题就很大，一天打印上千个日志文件，大小几十G，这根本没有日志的意义了，纯属浪费资源。

```xml
<logback.version>1.2.12</logback.version>
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>${logback.version}</version>
</dependency>

```

1. 关于`totalSizeCap`未生效问题，logback也经历过几个版本的迭代。
   \- logback1.2.3中是`{1,3}`
   \- logback1.2.x中是`{1,5}`
   \- logback 1.3.x 使用正则匹配所有数字`\d`

# 五、参考

 https://jira.qos.ch/browse/LOGBACK-1500
 https://jira.qos.ch/browse/LOGBACK-1297
