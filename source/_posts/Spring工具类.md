---
title: Spring工具类
top: false
cover: false
toc: true
mathjax: true
categories:
  - Spring - Spring基础 - Spring工具类
tags:
  - Spring - Spring基础 - Spring工具类
date: 2020-12-12 21:46:07
password:
summary:
---

# 1 Spring工具

## 1 IdGenerator唯一键生成器 UUID

![image-20201212215449169](image-20201212215449169.png)

### 1 **JdkIdGenerator** 

> JDK内置UUID生成器

### 2 **AlternativeJdkIdGenerator** 

> Spring提供的UUID生成器，替换调用`UUID#randomUUID()`。它提供了一个更好、更高性能的表现

### 3 **SimpleIdGenerator**

> 类似于自增的Id生成器。每调用一次，自增1

## 2 Assert 断言工具类

> Assert断言工具类，通常用于数据合法性检查。Assert断言工具类，通常用于数据合法性检查.

```java
Assert.notNull(Object object, "object is required")  
```

## 3 PathMatcher 路径匹配器

> Spring提供的实现：`AntPathMatcher`：Ant路径匹配规则

1 AntPathMatcher可以匹配@RequestMapping，也可以用来匹配各种字符串，包括文件路径

### **1 匹配方式**

* `?`匹配单字符
* `*`匹配0或者任意数量的字符
* `**` 匹配0或者更多目录

| URL路径     | 说明              |
| ----------- | ----------------- |
| /test/*.x   | 匹配以.x结尾的    |
| /test/a??le | 类似于数据库的`_` |
| /test/**/   |                   |

```java
assertFalse(pathMatcher.match("test*", "test/")); //注意这里是false 因为路径不能用*匹配
assertFalse(pathMatcher.match("test*", "test/t")); //这同理
//这个需要特别注意：{}里面的相当于Spring MVC里接受一个参数一样，所以任何东西都会匹配的
assertTrue(pathMatcher.match("/{bla}.*", "/testing.html"));
assertFalse(pathMatcher.match("/{bla}.htm", "/testing.html")); //这样就是false了
```

**最长匹配规则（has more characters）**	

> ​		即越精确的模式越会被优先匹配到。例如，URL请求/app/dir/file.jsp，现在存在两个路径匹配模式/**/*.jsp和/app/dir/*.jsp，那么会根据模式/app/dir/*.jsp来匹配。

### 2  匹配文件路径

> ​		AntPathMatcher不仅可以匹配URL路径，也可以匹配文件路径。但是需要注意AntPathMacher有构造参数，传递分割参数`pathSeparator`若不传值默认为`/ `，对于操作系统来说，需要根据不同的操作系统来传递文件分割符，以此来防止匹配路径错误。

```java
AntPathMatcher matcher=new AntPathMatcher(File.separator);
AntPathMatcher matcher = new AntPathMatcher(System.getProperty("file.separator"));
```

## 4 ConcurrentReferenceHashMap

> JDK也为我们提供了`WeakHashMap`来使用。但是，但是它并不是线程安全的，因此刚好Spring给我们功提供这个工具类：`ConcurrentReferenceHashMap`满足了我们对线程安全的弱、软引用的需求。

```java
@Test
public void test() throws InterruptedException {
    String key = new String("key");
    String value = new String("val");
    Map<String, String> map = new ConcurrentReferenceHashMap<>(8, ConcurrentReferenceHashMap.ReferenceType.WEAK);
    map.put(key, value);
    System.out.println(map); //{key=val}
    key = null;
    System.gc();

    //等待一会 确保GC能够过来
    TimeUnit.SECONDS.sleep(5);
    System.out.println(map); //{}
}
```

> 我们发现当我们把key设置为null后，垃圾回收器会回收掉这部分内存。这就是弱、虚引用的作用，主要用来防止OOM。

## 5 DefaultPropertiesPersister

> 这个类本身没有什么特别的，就是代理了JDK的Properties类而已。但写到此处是觉得Spring优秀就优秀在它强大的`对扩展开放的原则`体现。
>
> 比如如果我们需要对配置文件进行解密（比如数据库连接密码不能明文），这些操作通过复写这些扩展类的某些方法来做，将特别的优雅。

[spring启动时，解密配置文件的密文](https://www.xuebuyuan.com/1924773.html)

## 6 DigestUtils

> 可以对字节数组、InputStream流生成摘要。10禁止或者16进制都行

## 7 FastByteArrayOutputStream

> ​		增强版的`ByteArrayOutputStream`实底层原理是Spring采用了一个LinkedList来作为缓冲区,而`ByteArrayOutputStream`直接使用的字节数组。这样每一次扩容中分配一个数组的空间，并当该数据放入到List中。相当于批量的操作，而ByteArrayOutputStream内部实现为一个数组每一次扩容需要重新分配空间并将数据复制到新数组中。效率高下立判了~因此推荐使用

```java
	// The buffers used to store the content bytes
	private final LinkedList<byte[]> buffers = new LinkedList<>();
```

## 8 FileCopyUtils、FileSystemUtils、StreamUtils

> ​		都是操作文件、操作流的一些工具类。这里就不做过多的介绍了，因为还是比较简单的。

## 9 LinkedCaseInsensitiveMap ，LinkedMultiValueMap

### 1 LinkedCaseInsensitiveMap 

> ​		不区分大小写的有序map;底层代理了LinkedHashMap，因此它能保证有序。此Map的意义在于：在编写比如MaBatis这种类似的自动封装框架的时候，特备有用。
>
> ​		**数据库本身对大小写不敏感（例如mysql），但是创建表格的时候，数据库里面字段都会默认小写，所以MyBatis映射的时候，key也会映射成小写，可以用LinkedCaseInsensitiveMap（key值不区分大小写的LinkedMap）来处理。**

**注意：**

> LinkedCaseInsensitiveMap的key一定是String

### 2 LinkedMultiValueMap

> 见名之意，一个key对应多个value。废话不多说，直接感受一把就知道了。没有Apache Common提供的好用。但是一般来说也足够用了~

```java
   @Test
    public void fun1() {
        //用Map接的时候  请注意第二个泛型 是个List哦
        //Map<String, List<Integer>> map = new LinkedMultiValueMap<>();
        LinkedMultiValueMap<String, Integer> map = new LinkedMultiValueMap<>();

        //此处务必注意，如果你还是用put方法  那是没有效果的 同一个key还是会覆盖
        //map.put("a", Arrays.asList(1));
        //map.put("a", Arrays.asList(1));
        //map.put("a", Arrays.asList(1));
        //System.out.println(map); //{a=[1]}

        //请用add方法
        map.add("a", 1);
        map.add("a", 1);
        map.add("a", 1);
        System.out.println(map); //{a=[1, 1, 1]}
    }
```

## 10 PropertyPlaceholderHelper

> ​	作用：将字符串里的占位符内容，用我们配置的properties里的替换。这个是一个单纯的类，没有继承没有实现，没有依赖Spring框架其他的任何类。
>
> 我们自定义支持的表达式的时候，我们也可以高大上的说，我的格式支持spel表达式`SpelExpressionParser`。更加的灵活了。
>
> @Value注解或者其余spel表达式的东西，都得依赖于这个来做。

```java
public static void main(String[] args) throws Exception {
    String a = "{name}{age}{sex}";
    String b = "{name{age}{sex}}";
    PropertyPlaceholderHelper propertyPlaceholderHelper = new PropertyPlaceholderHelper("{", "}");
    InputStream in = new BufferedInputStream(new FileInputStream(new File("D:\\work\\remotegitcheckoutproject\\myprojects\\java\\boot2-demo1\\src\\main\\resources\\application.properties")));
    Properties properties = new Properties();
    properties.load(in);

    //==============开始解析此字符串==============
    System.out.println("替换前:" + a); //替换前:{name}{age}{sex}
    System.out.println("替换后:" + propertyPlaceholderHelper.replacePlaceholders(a, properties)); //替换后:wangzha18man
    System.out.println("====================================================");
    System.out.println("替换前:" + b); //替换前:{name{age}{sex}}
    System.out.println("替换后:" + properties); //替换后:love  最后输出love，证明它是从内往外一层一层解析的
}
```
## 11 SpringVersion、SpringBootVersion 

> 获取当前Spring版本号

## 12 ReflectionUtils 反射工具类

> 反射在容器中使用是非常频繁的了，这些方法一般在Spring框架内部使用。

### 1 变量

> 该方法是从类里面找到字段对象(private以及父类的都会找哟，但是static的就不会）

```java
public static Field findField(Class<?> clazz, String name);
public static Field findField(Class<?> clazz, @Nullable String name, @Nullable Class<?> type);
public static void setField(Field field, @Nullable Object target, @Nullable Object value);
// 测试代码
public static void main(String[] args) {
    Person person = new Person("fsx", 18);
    Field field = ReflectionUtils.findField(Person.class, "name");
    field.setAccessible(true); //注意，如果是private的属性，请加上这一句，否则抛出异常：can not access a member of class com.fsx.boot2demo1.bean.Person with modifiers "private"

    System.out.println(ReflectionUtils.getField(field, person)); //fsx
}
```

### 2 方法

> 也是一样非常的强大。private、父类方法都能获取到。

```java
public static Method findMethod(Class<?> clazz, String name);
public static Method findMethod(Class<?> clazz, String name, @Nullable Class<?>... paramTypes);
public static Object invokeMethod(Method method, @Nullable Object target); //这个不需要自己处理异常哦
public static Object invokeMethod(Method method, @Nullable Object target, @Nullable Object... args);

public static void main(String[] args) {
    Person person = new Person("fsx", 18);
    System.out.println(ReflectionUtils.findMethod(Person.class, "clone")); //protected native java.lang.Object java.lang.Object.clone() throws java.lang.CloneNotSupportedException
    System.out.println(ReflectionUtils.findMethod(Person.class, "getName")); //public java.lang.String com.fsx.boot2demo1.bean.Person.getName()
    System.out.println(ReflectionUtils.findMethod(Person.class, "setName", String.class)); //public void com.fsx.boot2demo1.bean.Person.setName(java.lang.String)
    System.out.println(ReflectionUtils.findMethod(Person.class, "privateMethod")); //private void com.fsx.boot2demo1.bean.Person.privateMethod()
}
```

### 3 异常

```java
public static void handleReflectionException(Exception ex);
public static void rethrowRuntimeException(Throwable ex);

boolean declaresException(Method method, Class<?> exceptionType); //判断一个方法上是否声明了指定类型的异常
boolean isPublicStaticFinal(Field field); //判断字段是否是public static final的
boolean isEqualsMethod(Method method); //判断该方法是否是equals方法
boolean isHashCodeMethod(Method method);
boolean isToStringMethod(Method method);
boolean isObjectMethod(Method method); //判断该方法是否是Object类上的方法

public static void makeAccessible(Field field); //将一个字段设置为可读写，主要针对private字段
void makeAccessible(Method method);
void makeAccessible(Constructor<?> ctor);
```

> 在AopUtils中也有这几个isXXX方法，是的，其实AopUtils中的isXXX方法就是调用的ReflectionUtils的这几个方法的；所以可见此工具类的强大

```java
public static void main(String[] args) {
    Person person = new Person("fsx", 18);
    Method[] allDeclaredMethods = ReflectionUtils.getAllDeclaredMethods(Person.class);
    //我们发现  这个方法可以把所有的申明的方法都打印出来。包含private和父类的   备注：重复方法都会拿出来。比如此处的toString方法 子类父类的都有
    for (Method method : allDeclaredMethods) {
        System.out.println(method);
    }

    System.out.println("------------------------------------");

    //针对于上面的结果过滤。只会保留一个同名的方法（保留子类的）
    Method[] uniqueDeclaredMethods = ReflectionUtils.getUniqueDeclaredMethods(Person.class);
    for (Method method : uniqueDeclaredMethods) {
        System.out.println(method);
    }

}
```

4 方法

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019020220230677.png)

> 针对指定类型上的所有方法，依次调用MethodCallback回调；看个源码就知道这个方法的作用：

```java
public static void doWithLocalMethods(Class<?> clazz, MethodCallback mc) {
    Method[] methods = getDeclaredMethods(clazz);
    for (Method method : methods) {
        try {
            mc.doWith(method);
        }catch (IllegalAccessException ex) {
            throw new IllegalStateException("...");
        }
    }
}

```

> 其实实现很简单，就是得到类上的所有方法，然后执行回调接口；这个方法在Spring针对bean的方法上的标签处理时大量使用，比如`@Init，@Resource，@Autowire`等标签的预处理；

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190202202739940.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Y2NDEzODU3MTI=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190202202655814.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190202202655814.png)

## 13 ResourceUtils

> Spring 提供了一个 ResourceUtils 工具类，它支持“**classpath:**”和“**file:**”的地址前缀，它能够从指定的地址加载文件资源。（其实还支持“jar:和war:前缀”）

```java
public abstract class ResourceUtils {

	public static final String CLASSPATH_URL_PREFIX = "classpath:";
	public static final String FILE_URL_PREFIX = "file:";
	public static final String JAR_URL_PREFIX = "jar:";
	public static final String WAR_URL_PREFIX = "war:";
	...
	// 是否是一个URL
	public static boolean isUrl(@Nullable String resourceLocation) {
		if (resourceLocation == null) {
			return false;
		}
		// 对classpath:进行了特殊的照顾
		if (resourceLocation.startsWith(CLASSPATH_URL_PREFIX)) {
			return true;
		}
		try {
			new URL(resourceLocation);
			return true;
		}
		catch (MalformedURLException ex) {
			return false;
		}
	}
	public static URL getURL(String resourceLocation) throws FileNotFoundException {...}
	public static File getFile(String resourceLocation) throws FileNotFoundException {...}
	public static File getFile(URL resourceUrl) throws FileNotFoundException {...}
	public static File getFile(URI resourceUri) throws FileNotFoundException {...}
	public static File getFile(URI resourceUri, String description) throws FileNotFoundException {...}
	
	// 判断
	public static boolean isFileURL(URL url) {
		// 均是和protocol 这个协议有关的~~~
		String protocol = url.getProtocol();
		return (URL_PROTOCOL_FILE.equals(protocol) || URL_PROTOCOL_VFSFILE.equals(protocol) ||
				URL_PROTOCOL_VFS.equals(protocol));
	}
	public static boolean isJarURL(URL url) {...}
	// @since 4.1
	public static boolean isJarFileURL(URL url) {...}
	// URL和URI的转换
	public static URI toURI(URL url) throws URISyntaxException {
		return toURI(url.toString());
	}
	public static URI toURI(String location) throws URISyntaxException {
		return new URI(StringUtils.replace(location, " ", "%20"));
	}
}
```

**例如：**

```java
public static void main(String[] args) throws FileNotFoundException {
    File file = ResourceUtils.getFile("classpath:application.properties");
    System.out.println(file); //D:\work\remotegitcheckoutproject\myprojects\java\boot2-demo1\target\classes\application.properties

    System.out.println(ResourceUtils.isUrl("classpath:application.properties")); //true

    //注意此处输出的路径为正斜杠 ‘/’的  上面直接输出File是反斜杠的（和操作系统相关）
    System.out.println(ResourceUtils.getURL("classpath:application.properties")); //file:/D:/work/remotegitcheckoutproject/myprojects/java/boot2-demo1/target/classes/application.properties
}
```
> 备注：若你在使用过程中，发现ResourceUtils.getFile()死活都找不到文件的话，那我提供一个建议：是否是在jar包内使用了此工具类，一般不建议在jar包内使用。另外，提供一个代替方案：可解决大多数问题：

```java
public static void main(String[] args) {
    ClassPathResource resource = new ClassPathResource("application.properties");
    System.out.println(resource); //class path resource [application.properties]
}
```

## 14 SerializationUtils

> 这个工具就不做过多解释了。提供了两个方法：对象<–>二进制的相互转化。（基于源生JDK的序列化方式）

```java
public static byte[] serialize(@Nullable Object object);
public static Object deserialize(@Nullable byte[] bytes);

```

## 15 SocketUtils

> 提供给我们去系统找可用的Tcp、Udp端口来使用。有的时候确实还蛮好用的，必进端口有时候不用写死了，提高灵活性

```java
public static void main(String[] args) {
    System.out.println(SocketUtils.PORT_RANGE_MAX); //65535 最大端口号
    System.out.println(SocketUtils.findAvailableTcpPort()); //45569 随便找一个可用的Tcp端口 每次执行值都不一样哦
    System.out.println(SocketUtils.findAvailableTcpPort(1000, 2000)); //1325 从指定范围内随便找一个端口

    //找一堆端口出来  并且是排好序的
    System.out.println(SocketUtils.findAvailableTcpPorts(10, 1000, 2000)); //[1007, 1034, 1287, 1483, 1494, 1553, 1577, 1740, 1963, 1981]

    //UDP端口的找寻 同上
    System.out.println(SocketUtils.findAvailableUdpPort()); //12007
}
```
> 其长度都是16个bit，所以端口号范围是0到(2^16-1)，即 0到 65535。其中0到1023是IANA规定的系统端口，即系统保留窗口。**例如HTTP为80端口，DNS服务为53端口**。下面列出常用tcp和udp重要协议端口号，供以参考

## 16 StringUtils

### 1 字符串判定

```java
//判断类：
// boolean isEmpty(Object str):字符串是否为空或者空字符串：""
// boolean hasLength(CharSequence str):字符串是否为空，或者长度为0
// boolean hasText(String str):字符串是否有内容（不为空，且不全为空格）
assertFalse(StringUtils.hasText("   "));
// boolean containsWhitespace(String str):字符串是否包含空格
assertTrue(StringUtils.containsWhitespace("a b"));
```

### 2 字符串头尾操作

```java
//字符串头尾操作
// String trimWhitespace(String str):去掉字符串前后的空格
assertEquals("abc", StringUtils.trimWhitespace(" abc "));
// String trimAllWhitespace(String str):去掉字符串中所有的空格
assertEquals("abc", StringUtils.trimAllWhitespace(" a b c "));
// String trimLeadingWhitespace(String str):去掉字符串开头的空格
// String trimTrailingWhitespace(String str):去掉字符串结束的空格

// String trimLeadingCharacter(String str, char leadingCharacter):去掉字符串开头的指定字符；
// String trimTrailingCharacter(String str, char trailingCharacter):去掉字符串结尾的指定字符；

// boolean startsWithIgnoreCase(String str, String prefix):
// 判断字符串是否以指定字符串开头，忽略大小写
assertTrue(StringUtils.startsWithIgnoreCase("abcd", "AB"));
// boolean endsWithIgnoreCase(String str, String suffix):
// 判断字符串是否以指定字符串结尾，忽略大小写
```

### 3 `文件操作`

> 文件路径名称相关操作，是针对文件名，文件路径，文件后缀等常见文件操作中需要用到的方法进行封装；

```java
// String unqualify(String qualifiedName):
// 得到以.分割的最后一个值，可以非常方便的获取类似类名或者文件后缀
assertEquals("java", StringUtils.unqualify("cn.wolfcode.java"));
assertEquals("java", StringUtils.unqualify("cn/wolfcode/Hello.java"));

// String unqualify(String qualifiedName, char separator):
// 得到以给定字符分割的最后一个值，可以非常方便的获取类似文件名
assertEquals("Hello.java", StringUtils.unqualify("cn/wolfcode/Hello.java", File.separatorChar));

// String capitalize(String str):首字母大写
assertEquals("Wolfcode", StringUtils.capitalize("wolfcode"));
// String uncapitalize(String str):取消首字母大写（首字母小写）
assertEquals("java", StringUtils.uncapitalize("Java"));

// String getFilename(String path):获取文件名,就不需要再使用FilenameUtils
assertEquals("myfile.txt",StringUtils.getFilename("mypath/myfile.txt"));
// String getFilenameExtension(String path):获取文件后缀名
assertEquals("txt",StringUtils.getFilenameExtension("mypath/myfile.txt"));
// String stripFilenameExtension(String path):截取掉文件路径后缀
assertEquals("mypath/myfile", StringUtils.stripFilenameExtension("mypath/myfile.txt"));

// String applyRelativePath(String path, String relativePath):
// 找到给定的文件，和另一个相对路径的文件，返回第二个文件的全路径
// 打印：d:/java/wolfcode/other/Some.java
System.out.println(StringUtils.applyRelativePath("d:/java/wolfcode/Test.java", "other/Some.java"));
// 但是不支持重新定位绝对路径和上级目录等复杂一些的相对路径写法：
// 仍然打印：d:/java/wolfcode/../other/Some.java
System.out.println(StringUtils.applyRelativePath("d:/java/wolfcode/Test.java", "../other/Some.java"));

// String cleanPath(String path): =====这个方法非常的重要
// 清理文件路径,这个方法配合applyRelativePath就可以计算一些简单的相对路径了
// 打印:d:/java/other/Some.java
System.out.println(StringUtils.cleanPath("d:/java/wolfcode/../other/Some.java"));
// 需求：获取d:/java/wolfcode/Test.java相对路径为../../other/Some.java的文件全路径：
// 打印：d:/other/Some.java
System.out.println(StringUtils.cleanPath(StringUtils.applyRelativePath( "d:/java/wolfcode/Test.java", "../../other/Some.java")));

// boolean pathEquals(String path1, String path2):
// 判断两个文件路径是否相同，会先执行cleanPath之后再比较
assertTrue(StringUtils.pathEquals("d:/wolfcode.txt","d:/somefile/../wolfcode.txt"));

```

### **4 字符串和子串的操作**

```java
// boolean substringMatch(CharSequence str, int index, CharSequence
// substring):判断从指定索引开始，是否匹配子字符串
assertTrue(StringUtils.substringMatch("aabbccdd", 1, "abb"));

// int countOccurrencesOf(String str, String sub):判断子字符串在字符串中出现的次数
assertEquals(4, StringUtils.countOccurrencesOf("ababaabab", "ab"));

// String replace(String inString, String oldPattern, String
// newPattern):在字符串中使用子字符串替换
assertEquals("cdcdacdcd", StringUtils.replace("ababaabab", "ab", "cd"));

// String delete(String inString, String pattern):删除所有匹配的子字符串；
assertEquals("a", StringUtils.delete("ababaabab", "ab"));

// String deleteAny(String inString, String charsToDelete):删除子字符串中任意出现的字符
assertEquals("", StringUtils.deleteAny("ababaabab", "bar"));

// String quote(String str) :在字符串前后增加单引号,比较适合在日志时候使用；
assertEquals("'hello'", StringUtils.quote("hello"));

```

### 5 **和Locale相关的一些字符串操作**

```java
// Locale parseLocaleString(String localeString):
// 从本地化字符串中解析出本地化信息，相当于Locale.toString()的逆向方法
assertEquals(Locale.CHINA, StringUtils.parseLocaleString("zh_CN"));

// @deprecated as of 5.0.4, in favor of {@link Locale#toLanguageTag()}
// String toLanguageTag(Locale locale):把Locale转化成HTTP中Accept-Language能接受的本地化标准；
// 比如标准的本地化字符串为：zh_CN，更改为zh-CN
System.out.println(StringUtils.toLanguageTag(StringUtils.parseLocaleString("zh_CN")));

```

### 6 **字符串和Properties**

```java
//Properties splitArrayElementsIntoProperties(String[] array, String delimiter):
// 把字符串数组中的每一个字符串按照给定的分隔符装配到一个Properties中
String[] strs=new String[]{"key:value","key2:中文"};
Properties ps=StringUtils.splitArrayElementsIntoProperties(strs, ":");
//打印输出：{key=value, key2=中文}
System.out.println(ps);

//Properties splitArrayElementsIntoProperties(String[] array, String delimiter, String charsToDelete)
//把字符串数组中的每一个字符串按照给定的分隔符装配到一个Properties中,并删除指定字符串，比如括号之类的；

```

7 **`字符串和数组之间的基本操作（重要）`**

```java
// String[] addStringToArray(String[] array, String str):把一个字符串添加到一个字符串数组中
// 打印：[a, b, c, d]
System.out.println(Arrays.toString(StringUtils
        .addStringToArray(new String[] { "a", "b", "c" }, "d")));

// String[] concatenateStringArrays(String[] array1, String[]array2):连接两个字符串数组
//打印：[a, b, c, a, b, c, d]
System.out.println(Arrays.toString(StringUtils.concatenateStringArrays(
        new String[] { "a", "b", "c" },
        new String[] { "a", "b", "c","d" })));

//String[] mergeStringArrays(String[] array1, String[] array2)：连接两个字符串数组，去掉重复元素
//打印：[a, b, c, d]
System.out.println(Arrays.toString(StringUtils.mergeStringArrays(
        new String[] { "a", "b", "c" },
        new String[] { "a", "b", "c","d" })));

//String[] sortStringArray(String[] array):字符串数组排序
//打印：[a, b, c, d]
System.out.println(Arrays.toString(StringUtils.sortStringArray(new String[]{"d","c","b","a"})));

//String[] toStringArray(Collection<String> collection):把字符串集合变成字符串数组
//String[] toStringArray(Enumeration<String> enumeration):把字符串枚举类型变成字符串数组
//String[] trimArrayElements(String[] array):把字符串数组中所有字符串执行trim功能；
//String[] removeDuplicateStrings(String[] array):去掉给定字符串数组中重复的元素，能保持原顺序；
//String[] split(String toSplit, String delimiter):按照指定字符串分割字符串；
assertArrayEquals(new String[]{"wolfcode","cn"}, StringUtils.split("wolfcode.cn", "."));
//只分割第一次，打印：[www, wolfcode.cn]
System.out.println(Arrays.toString(StringUtils.split("www.wolfcode.cn", ".")));


//String[] tokenizeToStringArray(String str, String delimiters)
//会对每一个元素执行trim操作，并去掉空字符串
//使用的是StringTokenizer完成，
//打印[b, c, d]
System.out.println(Arrays.toString(StringUtils.tokenizeToStringArray("aa,ba,ca,da", "a,")));
//String[] tokenizeToStringArray(String str, String delimiters, boolean trimTokens, boolean ignoreEmptyTokens)
//后面两个参数在限定是否对每一个元素执行trim操作，是否去掉空字符串

//String[] delimitedListToStringArray(String str, String delimiter):分割字符串，会把delimiter作为整体分隔符
//打印：[a, b, c, da]
System.out.println(Arrays.toString(StringUtils.delimitedListToStringArray("aa,ba,ca,da", "a,")));
//String[] delimitedListToStringArray(String str, String delimiter, String charsToDelete)
//分割字符串，会把delimiter作为整体分隔符，增加一个要从分割字符串中删除的字符；

//String[] commaDelimitedListToStringArray(String str):使用逗号分割字符串
//是delimitedListToStringArray(str, ",")的简单方法

//Set<String> commaDelimitedListToSet(String str)：使用逗号分割字符串，并放到set中去重
//使用LinkedHashSet;

//String collectionToDelimitedString(Collection<?> coll, String delim, String prefix, String suffix)
//将一个集合中的元素，使用前缀，后缀，分隔符拼装一个字符串，前缀后后缀是针对每一个字符串的
String[] arrs=new String[]{"aa","bb","cc","dd"};
assertEquals("{aa},{bb},{cc},{dd}", StringUtils.collectionToDelimitedString(Arrays.asList(arrs),",","{","}"));

//String collectionToDelimitedString(Collection<?> coll, String delim)：集合变成指定字符串连接的字符串；
//是collectionToDelimitedString(coll, delim, "", "")的简写；
//String collectionToCommaDelimitedString(Collection<?> coll)：集合变成逗号连接的字符串；
//是collectionToDelimitedString(coll, ",");的简写；

//String arrayToDelimitedString(Object[] arr, String delim)：数组使用指定字符串连接；
//String arrayToCommaDelimitedString(Object[] arr)：使用逗号连接数组，拼成字符串；

```

> **StringUtils类中的方法其实真的还是很多，可能平时我们用的比较多的还是一些普通的方法，其实类似文件路径，文件名等相关操作，以前还会专门引入common-io的FilenameUtils等额外的工具类，原来在StringUtils中都有，而且根据其设计的这些方法，我们也能大概的猜出一些方法在Spring中哪些地方可能有用；**

## 17 SystemPropertyUtils 占位符解析工具类

> 该类依赖于上面已经说到的`PropertyPlaceholderHelper`来处理。本类只处理系统默认属性值
>
> 该工具类很简单，但是非常的使用。可以很好的利用起来，Spring内部也有非常之多的使用。
>
> 在平时的程序开发中，我们也经常会遇到一些痛点，比如规则引擎（公式）：`A=${B}+${C}+1`，这样我们只需要得到B和C的值，然后再放入公式计算即可
>
> 占位符在Spring、Tomcat、Maven里都有大量的使用。下面用简单的例子体验一把：

```java
public static String resolvePlaceholders(String text);

//如果标志位为true，则没有默认值的无法解析的占位符将保留原样不被解析 默认为false
public static String resolvePlaceholders(String text, boolean ignoreUnresolvablePlaceholders);

```

```java
public static void main(String[] args) {
    System.out.println(SystemPropertyUtils.resolvePlaceholders("${os.name}/logs/app.log")); //Windows 10/logs/app.log

    //备注：这里如果不传true，如果找不到app.root这个key就会报错哦。传true后找不到也原样输出
    System.out.println(SystemPropertyUtils.resolvePlaceholders("${app.root}/logs/app.log", true)); //${app.root}/logs/app.log
}

```

另外，`org.springframework.core.io.support.PropertiesLoaderUtils`也是Spring提供的加载`.properties`配置文件的重要工具类之一~

[A哥Spring工具类第二篇](https://blog.csdn.net/f641385712/article/details/89380067?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522160777993819724818085193%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=160777993819724818085193&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_v2~rank_v29-1-89380067.pc_v2_rank_blog_default&utm_term=%E5%B7%A5%E5%85%B7%E7%B1%BB&spm=1018.2118.3001.4450)