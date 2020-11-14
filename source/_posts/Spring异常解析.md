---
title: Spring异常解析
top: false
cover: false
toc: true
mathjax: true
date: 2020-11-15 01:13:11
password:
summary:
tags:
categories:
---
[TOC]

# 一、异常解析器使用

* **`@ExceptionHandler`**

> 仅能处理**`HandlerMethod`**方式的异常，基本上满足99.9%的日常开发需求

*  **`AbstractHandlerExceptionResolver`**

> HandlerResolver实现类，可以实现`排序`（实现了Ordered接口）与`条件过滤`，这里的条件过滤主要支持两种类型的条件过滤。首先是为特定的Handler实例支持setMappedHandlers，第二个是支持与Class＃setMappedHandlerClasses相关的特定子类的实例

* `DefaultHandlerExceptionResolver`

> 解析标准Spring MVC异常并将其转换为相应的HTTP状态代码，也就是说，如果HandlerMapping参数无法解析，或者其他未解析的参数都由于此参数而转换为相应的response.sendError（HttpServletResponse.SC_NOT_FOUND),用于处理试图

* **`ResponseEntityExceptionHandler`**

> 为了解决返回视图的问题，spring提供了一个返回ResponseEntity ResponseEntityExceptionHandler的解决方案。通过@ExpectionHandler和@ControlerAdvice一起处理ResponseEntityExceptionHandler。这是上述DefaultHandlerExceptionResolver的副本，需要继承此类，然后在该类上使用@ControllerAdvice，并且必须全局注册ExceptionHandlerExceptionResolver

*  **`ResponseStatusExceptionResolver`**

> **`@ResponseStatus`标注在异常类上此处理器才会处理**，而不是标注在处理方法上，或者所在类上，**所以一般用于自定义异常时使用**。

*  **`SimpleMappingExceptionResolver`**

> 顾名思义它就是通过**简单映射**关系来决定由**哪个错误视图**来处理当前的异常信息。它提供了多种映射关系可以使用：

* `ExceptionHandlerExceptionResolver`

> 它是一个会在**Class及Class的父类**中找出带有`@ExceptionHandler`注解的类，该类带有key为`Throwable`，value为`Method`的**缓存属性**，提供匹配效率。

**示例：**

## 1. 全局异常处理

> 全局异常处理，指的是处理整个应用程序的异常，包括`过滤器`与`Controller`

### GlobalExpectionHandler

```java
@RestController
public class GlobalExpectionHandler extends ResponseEntityExceptionHandler {

    /**
     * 未知错误处理
     * @return
     */
    @ExceptionHandler(value = Exception.class)
    public String handler() {
        return " unknow error";
    }


    /**
     * 兜底处理器，避免返回视图
     * @param ex
     * @param headers
     * @param status
     * @param request
     * @return
     */
    @Override
    protected ResponseEntity<Object> handleHttpRequestMethodNotSupported(HttpRequestMethodNotSupportedException ex, HttpHeaders headers, HttpStatus status, WebRequest request) {
        //...定义自己的处理逻辑
        return super.handleHttpRequestMethodNotSupported(ex, headers, status, request);
    }
```

### GlobalErrorController

```java
@RestController
public class GlobalErrorController extends BasicErrorController {
    private final ErrorAttributes errorAttributes;

    public GlobalErrorController(ErrorAttributes errorAttributes, ServerProperties serverProperties) {
        super(errorAttributes, serverProperties.getError());
        this.errorAttributes = errorAttributes;

    }

    @RequestMapping(consumes = MediaType.APPLICATION_JSON_VALUE, produces = MediaType.APPLICATION_JSON_VALUE)
    public ResponseEntity<Object> errorJson(HttpServletRequest request) {
        HttpStatus status = getStatus(request);
        if (status == HttpStatus.NO_CONTENT) {
            return new ResponseEntity<>(status);
        }
        return new ResponseEntity<>("error ", status);
    }

}
```

# 二、自定义异常解析器

```java
@Component
public class CustomHandlerExpectionResovle extends AbstractHandlerExceptionResolver {
    public CustomHandlerExpectionResovle() {
        super();
        // 仅对TestController类型的处理器做处理
        setMappedHandlerClasses(TestController.class);
        // 这里可以拿到对应的mappedHandlers 进行设置
//        setMappedHandlers();
        setOrder(Ordered.HIGHEST_PRECEDENCE);
    }

    @Override
    protected ModelAndView doResolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        System.out.println();
        return new ModelAndView();
    }

    @Override
    protected boolean shouldApplyTo(HttpServletRequest request, Object handler) {
        if (handler instanceof HandlerMethod) {
            return super.shouldApplyTo(request, ((HandlerMethod) handler).getBean());
        }
        return false;
    }
}
```





# 三、源码初探

## 1.

## 2.

# 四、总结

![image-20201115011418803](image-20201115011418803.png)

![image-20201115011638203](image-20201115011638203.png)

![image-20201115013053350](image-20201115013053350.png)