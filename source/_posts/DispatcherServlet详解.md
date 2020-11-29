---
title: DispatcherServlet详解
top: false
cover: false
toc: true
mathjax: true
categories:
  - 源代码 - SpringMvc源码
tags:
  - 源码 - SpringMvc
date: 2020-11-19 18:33:56
password:
summary:
---

[toc]



# DispatcherServlet 过程分析

## 一 原始Servlet到SpringMvc

> 在过滤器执行完成之后，将会进入Servlet组件的执行中。首先出现的Servlet的类为HttpServlet，该类为抽象类

1. 首先进入HttpServlet的service方法

```java
/**
* 从Filter转向Servlet的入口方法，方法负责把原始的Servlet请求与相应转化为基于HTTP的Servlet请求与相应，* 再调用重载的Serivce方法
*/
public void service(ServletRequest req, ServletResponse res)
    throws ServletException, IOException {

    HttpServletRequest  request;
    HttpServletResponse response;

    try {
        // 对两个参数进行转换,只有基于Servlet的Web容器,产生的HTTP请求与响应就是下面返回的对应类型,这里的转换是用来提供重载Service方法
        request = (HttpServletRequest) req;
        response = (HttpServletResponse) res;
    } catch (ClassCastException e) {
        throw new ServletException(lStrings.getString("http.non_http"));
    }
    // 调用重载的方法
    service(request, response);
}
```

2. 进入父类的service进行执行逻辑

> 其中GET方法请求处理有些特殊，进行了HTTP缓存规范判断，如判断请求头中的If—Modified—Since已达到直接从浏览器缓存中获取请求结果的目的。但前提是需要支持LastModified功能。在DIspatcherServlet中，GET请求缓存通过另外的途径来实现，这里的getLastModified方法固定返回-1，故固定执行doGet方法

```java

protected void service(HttpServletRequest req, HttpServletResponse resp)
    throws ServletException, IOException {

    String method = req.getMethod();
// 对请求的方法进行  判断,支持以下七种方法
    if (method.equals(METHOD_GET)) {
        long lastModified = getLastModified(req);
        if (lastModified == -1) {
            // servlet doesn't support if-modified-since, no reason
            // to go through further expensive logic
            // 如果不支持缓存策略,执行doGet方法逻辑
            doGet(req, resp);
        } else {
            long ifModifiedSince;
            try {
                ifModifiedSince = req.getDateHeader(HEADER_IFMODSINCE);
            } catch (IllegalArgumentException iae) {
                // Invalid date header - proceed as if none was set
                ifModifiedSince = -1;
            }
            if (ifModifiedSince < (lastModified / 1000 * 1000)) {
                // If the servlet mod time is later, call doGet()
                // Round down to the nearest second for a proper compare
                // A ifModifiedSince of -1 will always be less
                maybeSetLastModified(resp, lastModified);
                doGet(req, resp);
            } else {
                resp.setStatus(HttpServletResponse.SC_NOT_MODIFIED);
            }
        }

    } else if (method.equals(METHOD_HEAD)) {
        long lastModified = getLastModified(req);
        maybeSetLastModified(resp, lastModified);
        doHead(req, resp);

    } else if (method.equals(METHOD_POST)) {
        doPost(req, resp);

    } else if (method.equals(METHOD_PUT)) {
        doPut(req, resp);

    } else if (method.equals(METHOD_DELETE)) {
        doDelete(req, resp);

    } else if (method.equals(METHOD_OPTIONS)) {
        doOptions(req,resp);

    } else if (method.equals(METHOD_TRACE)) {
        doTrace(req,resp);

    } else {
        //
        // Note that this means NO servlet supports whatever
        // method was requested, anywhere on this server.
        // 进入此逻辑表示请求方法不被支持,会直接返回错误响应
        // 因为PATCH方法与其他无法识别的请求方法在FrameworkServlet中重写的该方法提供了支持,故此处SpringMvc 下将不会进入

        String errMsg = lStrings.getString("http.method_not_implemented");
        Object[] errArgs = new Object[1];
        errArgs[0] = method;
        errMsg = MessageFormat.format(errMsg, errArgs);

        resp.sendError(HttpServletResponse.SC_NOT_IMPLEMENTED, errMsg);
    }
}
```

> 3. 在HttpServlet的do*方法中，均返回固定的错误请求，错误状态码为405，请求方法不被支持。所以若要对对应的请求方法提供支持，子类必须重写父类对应的请求do\*系列方法。
>
>    在SpringMvc中，真是的Servlet实例为DispatcherServlet，其中其父类FrameworkServlet重写了HttpServlet中所有的do*方法。
>
>    因为在SpringMvc的设计中，可以根据@RequestMapping注解标记的方法条件，把请求根据请求方法分发到不同的处理器方法上，故最终所有的方法需要调用同一个分发逻辑，所以在FrameworkServlet中的do\*方法，可以看到都调用了同一个方法processRequest
>
>    其实在不同的do*系列方法中，根据HTTP请求方法的定义，会做一些默认的处理，如doHead方法会调用doGet方法，FrameworkServlet中并没有重写doHead，而是通过重写doGet实现doHead的内部功能。同时还有doOptions方法进行了Options类型请求方法的特有处理，doTrace方法包括了特有的Trace相关的跟踪操作。
>
>    在上述过程执行完成之后，SpringMvc 提供的Servlet组件与Web容器整合，在后续的过程中，将调用与Servlet无关的processRequest方法，并以此为入口，进入到SpringMvc框架对请求的处理中。上述过程是从原始的Web容器调用到MVC框架组建的过程。

## 二 DispatcherServlet请求入口

> 在原始的Web容器入口开始，一步步对请求进行处理，在执行到Servlet层之后，开始进入SpringMvc的请求处理组件。所有的请求处理入口都在Spring的DispatchServlet组件中，都是以processDispatch开头的。



### 1 处理请求方法

```java
	/**
	 * Process this request, publishing an event regardless of the outcome.
	 * <p>The actual event handling is performed by the abstract
	 * {@link #doService} template method.
	 * 处理请求方法，用于执行所有请求的处理
	 */
	protected final void processRequest(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
		// 记录当前时间，用于日志记录
		long startTime = System.currentTimeMillis();
        // 失败原因，如请求处理过程抛出异常，则用该变量进行记录
		Throwable failureCause = null;
		// 获取当前的本地上下文，存储为上一个本地化上下文变量，以便在此请求处理中使用新的本地化上下文，在使用完成后通过该变量恢复原始的本地化上下文
		LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();
		// 构建当前请求的本地化上下文
        LocaleContext localeContext = buildLocaleContext(request);
		// 同本地化上下文，这里先存储上一次请求的请求属性上下文，以便在此请求处理中使用新的请求属性上下文，在使用完成后根据该变量恢复原始的请求属性上下文
		RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
		// 构造当前请求属性上下文
        ServletRequestAttributes requestAttributes = buildRequestAttributes(request, response, previousAttributes);
		// 获取或创建当前请求的异步管理器，用于对异步相应结果提供支持
		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
        // 拦截器RequestBindingInterceptor的作用同上面构造上下文逻辑和下面初始化上下文持有器initContextHolders与重置上下文持有器restContextHolders的作用相同
        // 因为请求的处理与当前线程是异步的关系，所以在其他线程执行初始化操作时就需要使用执行注册的这个拦截器
		asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(), new RequestBindingInterceptor());
		// 初始化上下文持有器，包括请求上下文处理器LocaleContextHolder和本地化上下文持有器RequestContextHolder，修改这两个持有器所持有得上下文为新建上下文
		initContextHolders(request, localeContext, requestAttributes);

		try {
            // 初始化完成之后执行真正处理请求方法
			doService(request, response);
		}
		catch (ServletException | IOException ex) {
            // 拦截Servlet和Io异常记录后再度抛出
			failureCause = ex;
			throw ex;
		}
		catch (Throwable ex) {
            // 记录其他异常
			failureCause = ex;
            // 记录后抛出新的前台异常
			throw new NestedServletException("Request processing failed", ex);
		}

		finally {
            // 重置上下文持有器，重置为该方法前面的逻辑保存的原始上下文。在异步请求拦截器里面，重置上下文则把上下文设置为NULL，这里则设置之前的上下文
			resetContextHolders(request, previousLocaleContext, previousAttributes);
			if (requestAttributes != null) {
                // 标记请求完成，并执行一些请求完成的回调方法
				requestAttributes.requestCompleted();
			}
            // 日志
			logResult(request, response, failureCause, asyncManager);
            // 向当前应用上下文中发布请求被处理的时间，事件类为ServletRequestHandlerEvent其中包含一些请求的相关属性，如请求路径、请求方法等
			publishRequestHandledEvent(request, response, startTime, failureCause);
		}
	}
```

### 2  请求分发概述

> 在整个方法中，每一段逻辑都有其自身的目的，且整体从前到后都是按照请求处理的流程在执行。同时，每一段逻辑的处理都涉及一些Spring Mvc 的组件

1. 预处理多块请求
2. 获取请求处理器
3. 查询处理适配器
4. 处理HTTP缓存
5. 执行前置拦截器链
6. 处理适配器执行
7. 返回值视图名处理
8. 执行后置拦截器链
9. 处理返回值与响应
10. 执行完成拦截器链
11. 清理资源

```Java

/**
* 执行请求分发到处理器
* 处理器通过当前应用中初始化的HandlerMapping处理器映射列表按顺序获取处理器
* 通过当前应用中初始化的HandlerAdapter处理适配器列表，获取支持当前请求处理器的处理适配器
* 该方法可以处理所有的请求方法。对于一些特殊的请求方法如Option是等，响应需要做额外的一些适配操作，该适配器操作交给请求处理器与处理适配器的逻辑去处理
*/
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    // 定义一个已处理器请求，指向参数中的request，已处理请求后序可能改变
   HttpServletRequest processedRequest = request;
    // 定义处理器执行链，内部封装拦截器列表与处理器
   HandlerExecutionChain mappedHandler = null;
   // 是否是多块请求，默认为否
   boolean multipartRequestParsed = false;
	// 获取与当前请求想关联的异步管理器，用于执行异步操作
   WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
	// 整体放在try中，用于捕获处理过程中的所有异常
   try {
       // 用于保存处理器适配器执行后的返回结果
      ModelAndView mv = null;
       // 用于保存处理过程中发生的异常
      Exception dispatchException = null;
		// 嵌套一个try，内部获取真实的处理异常，在异常处理中还有可能发生异常，上层的try作用为拦截这层try异常处理中发生的异常
      try {
          // 检查多块请求，如果是多块请求，则返回一个新的请求，processRequest保存这个心得请求引用；否则返回原始请求
         processedRequest = checkMultipart(request);
         // 判断两者是否是同一个引用，如果是说明是多块请求，且已经处理。此变量为true
          multipartRequestParsed = (processedRequest != request);

         // Determine handler for the current request.
          // 获取可处理当前请求的请求处理器，通过HandlerMapping查找，请求处理器中封装了拦截器链和对应的处理器，可以是具体的处理器方法
         mappedHandler = getHandler(processedRequest);
         // 如果没有则执行没有处理器逻辑
          if (mappedHandler == null) {
              // 内部逻辑判断配置throwExceptionIfNoHandlerFound是否为true，如果为true则爆出异常，否则直接设置响应内容为404
              // 可以通过spring.mvc.throwExceptionIfNoHandlerFound设置其值，默认为false
            noHandlerFound(processedRequest, response);
            return;
         }

         // Determine handler adapter for the current request.
          // 根据当前请求的处理器获取支持该处理器的处理器适配器
         HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

         // Process last-modified header, if supported by the handler.
          // 单独处理last-modified请求头，用于判断请求内容是否修改，如果未修改直接返回，浏览器使用本地缓存
         String method = request.getMethod();
          // 只有get请求和head请求执行此判断
         boolean isGet = "GET".equals(method);
         if (isGet || "HEAD".equals(method)) {
             // 具体实现还是通过处理器适配器来实现的
             // 通过处理器适配器的getLastModified方法，传入请求与处理器，获取该请求对应内容的最后修改时间
             // 一般针对静态资源，返回静态资源的上一次修改时间，动态资源固定返回-1，表示不存在时间
            long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
            // 经过判断，如果最后修改时间在当前请求中浏览器缓存时间之前，则直接返回状态码304，表示未修改，浏览器可以直接使用本地缓存作为请求内容
             // 否则继续执行请求处理逻辑，lastMOdified为-1固定执行后序请求处理逻辑
             if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
               return;
            }
         }
	// 通过mappedHandler这个HandlerExecutionChain执行链的封装，链式执行其中所有拦截器的前置拦方法preHandler
         if (!mappedHandler.applyPreHandle(processedRequest, response)) {
             // 任意一个拦截器的前置方法返回了false，即提前结束请求的处理
            return;
         }

         // Actually invoke the handler.
          // 最终执行处理器适配器的处理方法，传入请求，响应与其对应的处理器，对请求进行处理。在这个处理中，最终调用到了请求对应的处理器方法
          // 执行的返回值是ModelAndView类型，封装了模型数据与视图，后序对此结果进行处理并根据其中的视图与模型返回响应内容
         mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
		// 如果异步处理开始，则直接返回，后序处理均通过异步执行
         if (asyncManager.isConcurrentHandlingStarted()) {
            return;
         }
		// 应用默认视图名，如果返回值得ModelAndVIew中不包含视图名，则根据请求设置默认视图名
         applyDefaultViewName(processedRequest, mv);
          // 请求处理正常完成，链式执行所有拦截器的postHandler方法。链式顺序与preHandler相反
         mappedHandler.applyPostHandle(processedRequest, response, mv);
      }
      catch (Exception ex) {
          // 发现异常保存异常
         dispatchException = ex;
      }
      catch (Throwable err) {
         // As of 4.3, we're processing Errors thrown from handler methods as well,
         // making them available for @ExceptionHandler methods and other scenarios.
          // 在Spring Mvc4.3 版本以后，添加了支持error类型异常的处理。Throwable的子类除了Exception就是Error
          // 可以通过@ExceptionHandler处理这种类型的异常
          // 封装为嵌套异常，以供异常处理逻辑使用
         dispatchException = new NestedServletException("Handler dispatch failed", err);
      }
       // 对上面的逻辑的执行结果进行处理，包括处理适配器的执行结果以及发生的异常处理等逻辑
      processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
   }
   catch (Exception ex) {
       // 外层try的catch，用于拦截对执行结果的处理过程processDispatchResult中发生的异常
       // 拦截后链执行拦截器链afterCompletion方法。在该方法内部判断mappedHandler是否为空，如果不为空，则执行triggerAfterCompletion方法
      triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
   }
   catch (Throwable err) {
       // 拦截Error类型的异常拦截后链式执行拦截器链的afterCompletion方法
      triggerAfterCompletion(processedRequest, response, mappedHandler,
            new NestedServletException("Handler processing failed", err));
   }
    // 资源处理
   finally {

      if (asyncManager.isConcurrentHandlingStarted()) {
		 //如果在异步请求执行中，则链式执行拦截器链中的afterConcurrentHandlingStarted方法，即针对异步请求的特殊处理
         // Instead of postHandle and afterCompletion
         if (mappedHandler != null) {
            mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
         }
      }
      else {
          
         // Clean up any resources used by a multipart request.
         // 如果是多块资源，则清理多块资源占用的系统资源，包括文件缓存等
          if (multipartRequestParsed) {
            cleanupMultipart(processedRequest);
         }
      }
   }
}
```

#### 1  预处理多块请求

> 在SpringMvc中，对于多块请求有特殊的处理，如@RequestPart绑定多块请求参数与多块请求文件等。要想实现这些特殊处理，就需要先对请求类型为多块请求执行预处理，在请求分发前就执行该操作，并替换后序使用的请求已预处理过的多块请求。

```java
/**
 * 如果请求是多块请求，则转化为多块请求的多装数据类型，并标记多块请求解析器为可用
 * Convert the request into a multipart request, and make multipart resolver available.
 * <p>If no multipart resolver is set, simply use the existing request.
 * 如果设置为多块请求解析器，则直接使用原始请求
 * @param request current HTTP request
 * @return the processed request (multipart wrapper if necessary) 如果是多块请求则返回请求的包装类型
 * @see MultipartResolver#resolveMultipart
 */
protected HttpServletRequest checkMultipart(HttpServletRequest request) throws MultipartException {
    // 如果多块请求解析器不为空，使用多块请求解析器判断是否是多块请求，如果是，则执行后面的逻辑
   if (this.multipartResolver != null && this.multipartResolver.isMultipart(request)) {
       // 如果当前请求已经是多块请求的包装类型，则打印日志，并且返回
      if (WebUtils.getNativeRequest(request, MultipartHttpServletRequest.class) != null) {
         if (request.getDispatcherType().equals(DispatcherType.REQUEST)) {
            logger.trace("Request already resolved to MultipartHttpServletRequest, e.g. by MultipartFilter");
         }
      }
       // 如果当前请求中包含异常（通过请求属性java.servlet.error.exception 有无判断） 则返回传入的参数请求
      else if (hasMultipartException(request)) {
         logger.debug("Multipart resolution previously failed for current request - " +
               "skipping re-resolution for undisturbed error rendering");
      }
       // 否则执行多块请求解析
      else {
         try {
             // 尝试通过多块请求解析器解析多块请求，并返回多块请求解析器包装过的多块请求
            return this.multipartResolver.resolveMultipart(request);
         }
         catch (MultipartException ex) {
             // 发生异常时，先判断是否存在异常，如果已经存在则打印日志，否则抛出改异常
            if (request.getAttribute(WebUtils.ERROR_EXCEPTION_ATTRIBUTE) != null) {
               logger.debug("Multipart resolution failed for error dispatch", ex);
               // Keep processing error dispatch with regular request handle below
            }
            else {
               throw ex;
            }
         }
      }
   }
   // If not returned before: return original request.
    // 如果多块请求解析器为空，或者不为多块请求，或者执行其他不解析多块请求的多级，则直接返回原请求
   return request;
}
```

**总结：**

> 默认情况下，多块请求解析器为StandardServletMultipartResolver 其判断请求是否为多块请求的依据是请求方法为POST，且请求的Content-Type以multipart开头
>
> 多块请求解析器除了提供判断是否是多块请求的方法 isMultipart与解析多块请求的方法resolveMultipart外，还提供了清理多块请求占用资源的方法cleanMultpart，在逻辑清理的时候会使用到该功能

#### 2 查找请求处理器

> 对于请求的处理，最终都是要通过处理器来执行，SpringMvc把请求处理器的查找与请求处理器的执行分离。在doDispatch方法中，执行请求处理器的查找，也是SpringMvc的核心操作

```java
/**
* 返回该请求对应的请求处理链
 * Return the HandlerExecutionChain for this request.
 * <p>Tries all handler mappings in order.
 * 按顺序查找全部处理器映射
 * @param request current HTTP request
 * @return the HandlerExecutionChain, or {@code null} if no handler could be found
 */
@Nullable
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
 	// 如果请求映射列表不为空
    if (this.handlerMappings != null) {
        // 遍历全部处理器映射
      for (HandlerMapping mapping : this.handlerMappings) {
          // 尝试执行当前处理器映射的获取处理器方法，获取与本次请求适配的处理器执行链
         HandlerExecutionChain handler = mapping.getHandler(request);
          // 不为空直接返回，即便有多个处理器执行链匹配，也只返回第一个，处理器映射排在前面的优先返回
         if (handler != null) {
            return handler;
         }
      }
   }
   return null;
}
```

**总结：**

> ​			@RequestMapping注解注册的处理器方法，其相关的处理器映射类为RequestMappingHandlerMapping，该映射器内部根据当前请求与所有的注解信息进行匹配，找到最佳匹配并封装为处理器执行链进行返回
>
> ​			这也正是SpringMvc高度封装的体现，把一个复杂的逻辑都封装为一个接口，不仅可以通过不同的接口实现来完成不同的映射查找功能，同时无论映射查找功能有多复杂，这里看到的代码结构仍然非常的清晰。在这里把抽象与封装的概念体现的淋漓尽致
>
> ​			同时注意，查找到的是处理器执行链，其中封装了最终执行的处理器，以及在执行处理器前后要执行的拦截器链，即把与改签请求匹配的所有的拦截器链式封装带处理器执行链中，以供后序使用

#### 3 获取处理器适配器

> 根据请求的类型不同，又需要使用不同的适配器去执行处理器，要想使用对应的处理器适配器执行处理器，需要获取对应的适配器

```java
/**
* 返回支持传入的处理器对象的处理器适配器
 */
protected HandlerAdapter getHandlerAdapter(Object handler) throws ServletException {
   if (this.handlerAdapters != null) {
       // 遍历处理适配器列表，调用supports方法，找到支持该处理器的适配器
      for (HandlerAdapter adapter : this.handlerAdapters) {
		// 按照顺序查找第一个支持的适配器被返回
         if (adapter.supports(handler)) {
            return adapter;
         }
      }
   }
   throw new ServletException("No adapter for handler [" + handler +
         "]: The DispatcherServlet configuration needs to include a HandlerAdapter that supports this handler");
}
```

**总结：**

> ​		处理器适配器不但是执行处理器，而且还有更大的作用即适配，用于把请求参数适配为处理器需要的参数，并执行处理器，之后再把处理器的返回值适配成ModelAndVIew统一的模型视图类型，用于后序操作做统一的处理。
>
> ​		对于@RequestMapping注解注册的处理器，类型为HandlerMethod，通过RequestMappingHandlerMapping返回。对弈该类型的处理器，其适配器为RequestMappingHandlerAdapter，内部处理逻辑极其复杂包括参数绑定和返回值处理。

#### 4 处理HTTP缓存

> ​		在获取处理适配器之后，额外添加了用于支持HTTP缓存的功能，因为该功能是上层统一功能，故直接在通用处理逻辑中天健
>
> ​		HTTP缓存是HTTP协议标准中的定义，用于提高HTTP请求的效率。允许浏览器对GET请求获取的资源进行缓存，如css，js等，图片资源，在下次浏览器发起相同的请求时，会携带本地缓存的资源时间或其他与资源相关的信息作为请求头来传递，如果通过If-Modified-Since请求头携带本地缓存的资源的最近一次修改时间。而在服务器的处理逻辑中，则根据该资源的最后修改时间与请求信息中的一些标志信息做对比判断，如果判断结果是请求方缓存仍然有效，则直接返回状态码304，表示服务端为此资源进行修改，可以直接使用缓存中的资源。通过这种缓存方式大大较少了请求内耗，对于客户端来说体验更加友好，对于服务端来说，压力会更小。
>
> ​		这一段逻辑是为了实现缓存目的而出现。首先缓存只支持GET和HEAD请求，再通过处理适配器的getLastModified方法获取当前请求与请求处理器对应的资源的最后一次修改的时间，一般对于静态资源有最后一次修改时间。之后调用new ServletWebRequest(request,response).checkNotModified(lastModified)检查当前请求中携带的缓存信息是否过期，如果未过期，则直接返回304，不在执行后序处理，如果已过期，直接按照原逻辑执行后序处理
>
> ​		对处理器方法来说，以这种方式获取的是动态资源，动态资源不进行缓存，所以lastModified的值为-1

#### 5 执行前置拦截器链

> ​		在查找请求处理器时，返回了请求处理器执行链，在其中封装了拦截器链，用于在执行请求逻辑前进行拦截、执行请求逻辑之后添加后置处理，还可以拦截所有请求处理，在所有处理完成时或发生异常时添加完成后操作

```java
/**
 * 执行已注册拦截器的处理前拦截方法
 * @return {@code true} 返回是否需要继续执行后序的处理，如果为false则表示请求被拦截器拦截，不再执行后序处理，否则继续执行后序处理
 */
boolean applyPreHandle(HttpServletRequest request, HttpServletResponse response) throws Exception {
   HandlerInterceptor[] interceptors = getInterceptors();
   if (!ObjectUtils.isEmpty(interceptors)) {
      for (int i = 0; i < interceptors.length; i++) {
         HandlerInterceptor interceptor = interceptors[i];
          // 如果返回false则直接停止执行，视为处理完成，出发拦截器完成后方法
         if (!interceptor.preHandle(request, response, this.handler)) {
            triggerAfterCompletion(request, response, null);
            return false;
         }
          //如果为true 拦截器索引为当前遍历的索引，用于提供给triggerAfterCompletion 使用
         this.interceptorIndex = i;
      }
   }
   return true;
}
```

**总结：**

> ​		在调用triggerAfterCompletion的时候，只会调用已经执行了preHandle方法的处理器（除了当前preHandle返回false的）。这就是索引interceptorIndex的作用

#### 6 处理器适配器执行

> ​		在完成上面的所有处理之后，会进入处理适配的执行逻辑，这里只是简单调用才处理器的handler方法，返回ModleAndView的值。
>
> ​		这个方法是整个请求处理最核心的方法，也是在调用了这个方法之后，最终才调用了处理器方法。所有对请求处理逻辑的封装都是在这个方法内部

#### 7 返回视图名处理

> ​		在处理适配器执行完成之后，返回了ModelAndView类型的返回值，在很多的情况下，返回值不包含视图，但对于响应结果来说，没有视图就无法产生响应，故此处会将执行默认的视图查找逻辑，以对此返回值应用默认视图。 	

```java
/**
 * Do we need view name translation?
 * 应用视图默认名称	 
 */
private void applyDefaultViewName(HttpServletRequest request, @Nullable ModelAndView mv) throws Exception {
    // 如果返回值不为空，并且不包含视图
   if (mv != null && !mv.hasView()) {
       // 根据逻辑获取默认的视图名称
      String defaultViewName = getDefaultViewName(request);
       // 如果获取的默认视图名称不为null，则将设置为ModelAndView的视图名
      if (defaultViewName != null) {
         mv.setViewName(defaultViewName);
      }
   }
   
}
// 通过视图名翻译器来根据请求获取视图名称
	@Nullable
	protected String getDefaultViewName(HttpServletRequest request) throws Exception {
		return (this.viewNameTranslator != null ? this.viewNameTranslator.getViewName(request) : null);
	}

```

**总结：**

> ​		在视图名的获取逻辑中又出现了新的组件ViewNameTranslator,其类型为RequestToViewNameTranslator该组件的作用是根据请求获取一个视图名称,此处该组件其默认实现为DefaultReqeustToViewNameTranlator
>
> ​		在该实现中，获取视图名的逻辑为获取请求路径，并拼接此实现中的前缀和后缀，作为默认的视图名，包括路径。默认的前缀、后缀都为空字符串，所以可直接视为视图名就是请求路径。  

#### 8 执行后置拦截器链

```java
/**
 * 执行拦截器链中的后置拦截
 */
void applyPostHandle(HttpServletRequest request, HttpServletResponse response, @Nullable ModelAndView mv)
      throws Exception {
	// 获取全部的拦截器
   HandlerInterceptor[] interceptors = getInterceptors();
   if (!ObjectUtils.isEmpty(interceptors)) {
      for (int i = interceptors.length - 1; i >= 0; i--) {
         HandlerInterceptor interceptor = interceptors[i];
         interceptor.postHandle(request, response, this.handler, mv);
      }
   }
}
```

#### 9 处理返回值与响应

> ​		通过方法processDispatchResult，处理前面过程中产生的分发结果，在无异常的时候主要的处理对象是处理适配器返回的ModelAndView。发生异常的时候，处理对象为产生的异常，异常处理结果为ModelAndView ，使用此ModelAndView继续执行后序处理

```java
/**
 * 用于处理适配器调用处理器后适配过的ModelAndView结果，或者发生异常时把异常处理为ModelAndView结果
 */
private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
      @Nullable HandlerExecutionChain mappedHandler, @Nullable ModelAndView mv,
      @Nullable Exception exception) throws Exception {
	// 标记是否为error视图
   boolean errorView = false;
	// 如果出现了异常
   if (exception != null) {
       // 如果异常类型为ModelAndViewDefiningException
      if (exception instanceof ModelAndViewDefiningException) {
 	 // 直接使用异常中封装的ModleAndView作为最终的ModleAndView结果
         mv = ((ModelAndViewDefiningException) exception).getModelAndView();
      }else {
          // 其他异常类型,先获取处理器
         Object handler = (mappedHandler != null ? mappedHandler.getHandler() : null);
          // 执行process处理器异常方法,获取了处理器异常结果后的得到的ModleAndView结果
          mv = processHandlerException(request, response, handler, exception);
          // 如果mv不为空,则说明返回了包含异常的视图,即返回的视图为异常视图
         errorView = (mv != null);
      }
   }

   // Did the handler return a view to render?
   //  如果视图与模型不为空,且视图与模型没有标记为被清理(被清理表示调用过ModelAndView的clear方法,清理后ModelAndView相当于Null)
   if (mv != null && !mv.wasCleared()) {
       // 视图与模型不为空时,执行渲染视图的操作
      render(mv, request, response);
      if (errorView) {
          // 如果是异常视图,渲染后需要清楚请求属性的异常信息
         WebUtils.clearErrorRequestAttributes(request);
      }
   }
   else {
      if (logger.isTraceEnabled()) {
         logger.trace("No view rendering, null ModelAndView returned.");
      }
   }
	// 如果异步请求处理已经开始,则直接返回结束执行
   if (WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
      // Concurrent handling started during a forward
      return;
   }
	// 如果处理器执行链不为空时，出发拦截器的完成后方法，这里的完成方法是在请求处理正常完成时执行的。还有异常时执行的完成后方法
   if (mappedHandler != null) {
      // Exception (if any) is already handled..
      mappedHandler.triggerAfterCompletion(request, response, null);
   }
}
```

> ​		在上面的处理过程中，有两个比较重要的方法。第一个是发生异常时，把异常处理为ModelAndView返回值逻辑processHandlerException。第二个是对返回的ModelAndVIew结果进行渲染的逻辑render

##### 1  处理器异常的处理方法

```java
/**
 * 通过处理器异常解析器来把产生的异常解析为一个错误视图与模型结果
 * @param request current HTTP request 请求
 * @param response current HTTP response 响应
 * @param handler 执行的处理器，也与可能为空，如果发生异常时还没有获取处理器。如在多块请求解析的时候发生异常
 * (for example, if multipart resolution failed)
 * @param ex  在请求处理的过程中发生的异常
 * @return a corresponding ModelAndView to forward to
 * @throws Exception if no error ModelAndView found
 */
@Nullable
protected ModelAndView processHandlerException(HttpServletRequest request, HttpServletResponse response,
      @Nullable Object handler, Exception ex) throws Exception {

   // 成功和错误响应可能使用不同的内容类型
   request.removeAttribute(HandlerMapping.PRODUCIBLE_MEDIA_TYPES_ATTRIBUTE);

   // Check registered HandlerExceptionResolvers...
   ModelAndView exMv = null;
    // 如果处理器异常解析器列表不为空
   if (this.handlerExceptionResolvers != null) {
      for (HandlerExceptionResolver resolver : this.handlerExceptionResolvers) {
          // 执行异常处理器拿到结果
         exMv = resolver.resolveException(request, response, handler, ex);
          // 如果不为空，则将此结果作为对异常处理后的ModelAndView结果使用，中断后序遍历动作
         if (exMv != null) {
            break;
         }
      }
   }
    // 如果返回的异常ModelAndView不为null
   if (exMv != null) {
       // 如果ModelAndView内部为空 即modle为空view为空
      if (exMv.isEmpty()) {
          // 设置异常属性到请求属性中
         request.setAttribute(EXCEPTION_ATTRIBUTE, ex);
         return null;
      }
  	// 如果异常的ModelAndView不包括视图
      if (!exMv.hasView()) {
          //采用doDispatch方法中相同的处理逻辑来根据请求获取默认的视图名称
         String defaultViewName = getDefaultViewName(request);
         if (defaultViewName != null) {
            exMv.setViewName(defaultViewName);
         }
      }
      if (logger.isTraceEnabled()) {
         logger.trace("Using resolved error view: " + exMv, ex);
      }
      else if (logger.isDebugEnabled()) {
         logger.debug("Using resolved error view: " + exMv);
      }
       // 暴露一些异常信息到请求的属性当中
      WebUtils.exposeErrorRequestAttributes(request, ex, getServletName());
      return exMv;
   }
	//如果没有异常处理器解析器，则原封不动的抛出原始异常，交给Web框架做处理
   throw ex;
}
```

> ​		在把异常解析为ModelAndView结果时，其核心是处理器异常解析器组件。
>
> ​		在processDispatchResult方法中，对结果进行统一的处理，进行渲染

##### 2 渲染

```java
/**
 * 对指定的ModelAndView进行渲染
 * 这一步是一个请求的处理过程中的最后一步，其中包含了通过视图名获取视图的逻辑
 *
 * @param mv the ModelAndView to render
 * @param request current HTTP servlet request
 * @param response current HTTP servlet response
 * @throws ServletException  视图不存在或者不能解析抛出
 * @throws Exception 渲染发生任何异常抛出
 */
protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
   // 通过Locale解析器获取请求对应的Locale
   Locale locale =
         (this.localeResolver != null ? this.localeResolver.resolveLocale(request) : request.getLocale());
    //设置获取的Locale为相应的Locale
   response.setLocale(locale);
	// 最终获取的视图
   View view;
    // 如果ModelAndView 中视图为视图名，则获取这个名字
   String viewName = mv.getViewName();
   if (viewName != null) {
      // 把名字解析为视图
      view = resolveViewName(viewName, mv.getModelInternal(), locale, request);
       // 视图为空的时候同样排除异常
      if (view == null) {
         throw new ServletException("Could not resolve view with name '" + mv.getViewName() +
               "' in servlet with name '" + getServletName() + "'");
      }
   }
   else {
     // 如果不是视图名，而直接是一个视图类型，则获取视图
      view = mv.getView();
       //视图为空时同样抛出异常
      if (view == null) {
         throw new ServletException("ModelAndView [" + mv + "] neither contains a view name nor a " +
               "View object in servlet with name '" + getServletName() + "'");
      }
   }
	//代理调用视图类的渲染方法
   // Delegate to the View object for rendering.
   if (logger.isTraceEnabled()) {
      logger.trace("Rendering view [" + view + "] ");
   }
   try {
       // 如果ModelAndView中的status部位为空，则设置为相应的状态码@ResponseStatus设置的状态码功能就是通过这里实现的
      if (mv.getStatus() != null) {
         response.setStatus(mv.getStatus().value());
      }
       // 通过视图的渲染方法，每种模板引擎，都有其对应的视图实现，视图渲染对应于模板引擎的渲染模板
      view.render(mv.getModelInternal(), request, response);
   }
   catch (Exception ex) {
      if (logger.isDebugEnabled()) {
         logger.debug("Error rendering view [" + view + "]", ex);
      }
      throw ex;
   }
}
```

其中第一个关键点是通过视图名解析视图的方法resloveViewName，该方法内容如下

```java
	@Nullable
	protected View resolveViewName(String viewName, @Nullable Map<String, Object> model,
			Locale locale, HttpServletRequest request) throws Exception {
	// 如果视图解析器列表不为空
		if (this.viewResolvers != null) {
			for (ViewResolver viewResolver : this.viewResolvers) {
                //将视图名称解析为视图
				View view = viewResolver.resolveViewName(viewName, locale);
				if (view != null) {
					return view;
				}
			}
		}
		return null;
	}

```

**总结：**

> ​			这里又通过新的最贱---视图解析器来对视图名称进行解析，得到最终想要的视图，不同的模板引擎有不同的视图解析器，例如Thymeleaf模板引擎，对应的视图解析器为ThymeleafViewResolver，解析逻辑则是通过配置spring.thymeleaf.prefix+视图名+spring.thymeleaf.suffix的形式，从classpath下查找视图资源，并解析为具体的ThymyleafView
>
> ​			获取具体的视图之后后续的处理核心就是视图渲染的方法执行。在视图渲染的方法执行过程中，通过Model对模板进行渲染，并吧渲染后的结果写入相应的输出流，最终返回给请求方。不同模板引擎对应着不同的视图，不同的视图，又有自身的渲染方法 

​    

#### 10  执行完拦截器链

```java
void triggerAfterCompletion(HttpServletRequest request, HttpServletResponse response, @Nullable Exception ex)
      throws Exception {
	// 获取全部拦截器
   HandlerInterceptor[] interceptors = getInterceptors();
   if (!ObjectUtils.isEmpty(interceptors)) {
      for (int i = this.interceptorIndex; i >= 0; i--) {
         HandlerInterceptor interceptor = interceptors[i];
          // 执行拦截器的afterCompletion 方法，放在try块中，以保证执行中不再向外抛出异常，因为执行此方法时可能已经发生异常了，而该方法的执行不会再对内部发生的异常进行捕获，避免覆盖上层的异常
         try {
            interceptor.afterCompletion(request, response, this.handler, ex);
         }
         catch (Throwable ex2) {
            logger.error("HandlerInterceptor.afterCompletion threw exception", ex2);
         }
      }
   }
}
```

#### 11 清理资源

> ​		在finally中，首先对异步请求处理执行了处理器执行链中的afterConcurrentHandlingStarted方法，为异步请求开始执行时添加一些逻辑。
>
> ​		除此之外，核心的清理方法是对多块请求的清理。如果是多块请求，且多块请求是被处理过的，则执行多块请求占用资源的清理方法

```java
/**
 * Clean up any resources used by the given multipart request (if any).
 * @param request current HTTP request
 * @see MultipartResolver#cleanupMultipart
 */
protected void cleanupMultipart(HttpServletRequest request) {
   if (this.multipartResolver != null) {
      MultipartHttpServletRequest multipartRequest =
            WebUtils.getNativeRequest(request, MultipartHttpServletRequest.class);
      if (multipartRequest != null) {
          // 执行多块请求数据的清理
         this.multipartResolver.cleanupMultipart(multipartRequest);
      }
   }
}
```

**总结 :**

> ​		对于默认实现的多块请求解析器StandardServletMultipartResolver,其清理的逻辑是遍历多块数据part，如果多块数据part为文件数据，则执行它们的删除操作，以清理多块数据占用的临时文件