---
title: Spring Bean基础
top: false
cover: false
toc: true
mathjax: true
categories:
  -  Spring编程核心思想
tags:
  -  Spring编程核心思想
date: 2022-04-06 20:44:39
password:
summary:
---

# 一、定义Spring Bean

## 1 什么是BeanDefinition

> BeanDefinition是SPring Framework 中定义Bean的配置元信息接口。

包含

* Bean
* Bean 行为配置元素。如作用域、自动绑定模式、生命周期回调
* 其他Bean引用。又可称作合作者（Collaborators）或者依赖（Dependencies）
* 配置设置，比如Bean的属性（Properties）



# 二、BeanDefinition元信息

| 属性（Property）         | 说明                                         |
| ------------------------ | -------------------------------------------- |
| Class                    | Bean全类名，必须是具体类，不能用抽象类或接口 |
| Name                     | Bean的名称或ID                               |
| Scope                    | Bean的作用域（singleton、prototype等 ）      |
| Constructor arguments    | Bean 构造器参数（用户依赖注入）              |
| Properties               | Bean属性设置（用于依赖注入）                 |
| Autowiring mode          | Bean 自动绑定模式（如：byName）              |
| Lazy initialization mode | Bean延迟初始化模式（延迟和非延迟）           |
| Initialization Method    | Bean 初始化回调方法名称                      |
| Destruction Method       | Bean销毁回调方法名称                         |

## 1 BeanDefinition 构建

### 1.1 通过BeanDefinitionBuilder构建

```java

BeanDefinitionBuilder builder = BeanDefinitionBuilder.genericBeanDefinition(User.class);
builder.addPropertyValue("age", 18);
builder.addPropertyValue("name", "张三");
final BeanDefinition beanDefinition = builder.getBeanDefinition();
// beanDefinition 并非bean的最终状态可以直接修改
```



### 1.2 通过AbstractBeanDefinition 以及派生类

```java
// 2. 通过AbstractBeanDefinition 以及其派生
GenericBeanDefinition genericBeanDefinition = new GenericBeanDefinition();
// 设置bean类型
genericBeanDefinition.setBeanClass(User.class);
// 通过MutablePropertyValues 批量操作属性
MutablePropertyValues propertyValues = new MutablePropertyValues();
propertyValues.addPropertyValue("age", 18);
propertyValues.addPropertyValue("name", "张三");
genericBeanDefinition.setPropertyValues(propertyValues);
```



# 三、命名Spring Bean

**Bean的名称**    

> 每个Bean拥有一个或多个标识符（identifiers），这些标识符在Bean所在容器必须是唯一的，通常，一个Bean仅有一个标识符，如果需要额外的 

> 在基于XML的配置元信息中，开发人员可用ID，或者name属性来规定Bean的标识符。通常Bean的标识符由字母组成，允许出现特殊字符。如果要想引入Bean的别名的话，可在name属性使用半角（“，”）或者分号来分割

> Bean 的 id 或 name 属性并非必须制定，如果留空的话，容器会为 Bean 自动生成一个唯一 的名称。Bean 的命名尽管没有限制，不过官方建议采用驼峰的方式，更符合 Java 的命名约 定。

**Bean 名称生成器（BeanNameGenerator）**

* 由 Spring Framework 2.0.3 引入，框架內建两种实现： DefaultBeanNameGenerator：默认通用BeanNameGenerator 实现
* AnnotationBeanNameGenerator：基于注解扫描的 BeanNameGenerator 实现，起始于 Spring Framework 2.5， 

> *With component scanning in the classpath, Spring generates bean names for unnamed components,* *following the rules described earlier: essentially, taking the simple class name and turning its* initial character to lower-case. However, in the (unusual) special case when there is more than one character and both the first and second characters are upper case, the original casing gets preserved. These are the same rules as defined by java.beans.Introspector.decapitalize (which Spring uses here).



# 四、 Spring Bean 的别名

**Bean 别名（Alias）的价值** 

* 复用现有的 BeanDefinition
* 更具有场景化的命名方法，比如：

```xml
<alias name="myApp-dataSource"alias="subsystemA-dataSource"/> 
<alias name="myApp-dataSource"alias="subsystemB-dataSource"/>
```



# 五、注册Spring Bean



**BeanDefinition 注册** 

*  XML 配置元信息 
  * **<bean name=”...” ... />** 

* Java 注解配置元信息 

  * **@Bean** 

  * **@Component** 

  * **@Import** 

* Java API 配置元信息 

* 命名方式：**BeanDefinitionRegistry#registerBeanDefinition(String,BeanDefinition)** 

* 非命名方式： **BeanDefinitionReaderUtils#registerWithGeneratedName(AbstractBeanDefinition,BeanDefinitionRegistry)** 

* 配置类方式：**AnnotatedBeanDefinitionReader#register(Class...)**
* 外部单例对象注册
  * Java API 配置元信息
    * **SingletonBeanRegistry#registerSingleton**

```java

/**
 * 注解 BeanDefinition 示例
 
 */
// 3. 通过 @Import 来进行导入
@Import(AnnotationBeanDefinitionDemo.Config.class)
public class AnnotationBeanDefinitionDemo {

    public static void main(String[] args) {
        // 创建 BeanFactory 容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
        // 注册 Configuration Class（配置类）
        applicationContext.register(AnnotationBeanDefinitionDemo.class);

        // 通过 BeanDefinition 注册 API 实现
        // 1.命名 Bean 的注册方式
        registerUserBeanDefinition(applicationContext, "mercyblitz-user");
        // 2. 非命名 Bean 的注册方法
        registerUserBeanDefinition(applicationContext);

        // 启动 Spring 应用上下文
        applicationContext.refresh();
        // 按照类型依赖查找
        System.out.println("Config 类型的所有 Beans" + applicationContext.getBeansOfType(Config.class));
        System.out.println("User 类型的所有 Beans" + applicationContext.getBeansOfType(User.class));
        // 显示地关闭 Spring 应用上下文
        applicationContext.close();
    }

    public static void registerUserBeanDefinition(BeanDefinitionRegistry registry, String beanName) {
        BeanDefinitionBuilder beanDefinitionBuilder = genericBeanDefinition(User.class);
        beanDefinitionBuilder
                .addPropertyValue("id", 1L)
                .addPropertyValue("name", "小马哥");

        // 判断如果 beanName 参数存在时
        if (StringUtils.hasText(beanName)) {
            // 注册 BeanDefinition
            registry.registerBeanDefinition(beanName, beanDefinitionBuilder.getBeanDefinition());
        } else {
            // 非命名 Bean 注册方法
            BeanDefinitionReaderUtils.registerWithGeneratedName(beanDefinitionBuilder.getBeanDefinition(), registry);
        }
    }

    public static void registerUserBeanDefinition(BeanDefinitionRegistry registry) {
        registerUserBeanDefinition(registry, null);
    }

    // 2. 通过 @Component 方式
    @Component // 定义当前类作为 Spring Bean（组件）
    public static class Config {

        // 1. 通过 @Bean 方式定义

        /**
         * 通过 Java 注解的方式，定义了一个 Bean
         */
        @Bean(name = {"user"})
        public User user() {
            User user = new User();
            user.setId(1L);
            user.setName("小");
            return user;
        }
    }


}

```



# 六、实例化 Spring Bean

# 七、初始化 Spring Bean

# 八、延迟初始化Spring Bean

# 九、销毁 Spring Bean

# 十、垃圾回收Spring Bean
