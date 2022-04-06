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

# 四、 Spring Bean 的别名

# 五、注册Spring Bean

# 六、实例化 Spring Bean

# 七、初始化 Spring Bean

# 八、延迟初始化Spring Bean

# 九、销毁 Spring Bean

# 十、垃圾回收Spring Bean
