---
title: SpringMVC 异步模式
top: false
cover: false
toc: true
mathjax: true
categories:
  - SpringMvc
  - 异步
tags:
  - SpringMvc
date: 2020-12-13 17:01:22
password:
summary:
---

# 1 SpringMVC异步模式

> Spring的异步请求模式是Spring3.2就推出了，它是基于Servlet3.0规范实现的，而此规范是2011年推出的。

## 1 SpringMVC的同步模式



![image-20201213174021230](image-20201213174021230.png)

> 此处需要明晰一个概念：比如tomcat，它既是一个web服务器，同时它也是个servlet后端容器（调ava后端服务），所以要区分清楚这两个概念。请求处理线程是有限的，宝贵的资源~（注意它和处理线程的区别）

## 2 **Callable**案例

1 请求处理线程：处理线程属于Web服务器线程，负责处理用户请求，采用线程池管理

2 异步线程：异步线程属于用户自定义线程，可采用线程池管理

#### 代码示例

```java
@GetMapping
private Callable<Object> testCallable() {
    System.out.println(Thread.currentThread().getName() + "主线程开始");
    Callable<Object> callable = new Callable<Object>() {
        @Override
        public Object call() throws Exception {
            System.out.println(Thread.currentThread().getName() + " 子线程开始");
            TimeUnit.SECONDS.sleep(5);
            System.out.println(Thread.currentThread().getName() + " 子线程结束");
            Map<String, Object> map = new HashMap<>();
            map.put("name", "ken");
            map.put("age", "35");
            map.put("local", "中国");
            return map;
        }
    };
    System.out.println(Thread.currentThread().getName() + "主线程结束");
    return callable;
}
```

#### 结果

> 前端等待5秒后拿到数据，注意：异步模式对前端来说，是无感知的，这是后端的一种技术。所以这个和我们自己开启一个线程处理，立马返回给前端是有非常大的不同的，需要注意~

由此我们可以看出，主线程早早就结束了（需要注意，**此时还并没有把response返回的，此处一定要注意**），真正干事的是子线程（交给`TaskExecutor`去处理的，后续分析过程中可以看到），它的大致的一个处理流程图可以如下：

![image-20201213174039201](image-20201213174039201.png)

这里能够很直接的看出：我们很大程度上提高了我们`请求处理线程`的利用率，从而肯定就提高了我们系统的吞吐量。

#### 结论

> 当Controller返回值是Callable的时候，Spring就会将Callable交给TaskExecutor去处理（一个隔离的线程池），与此同时将DispatcherServlet里的拦截器、Filter等等都马上退出线程，但是response仍然保持打开的状态，Callable线程处理完成后，Spring MVC讲请求重新派发给容器**（注意这里的重新派发，和拦截器密切相关）**。根据Callabel返回结果，继续处理（比如参数绑定、视图解析等等就和之前一样了）

[Spring文档介绍](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-callable)

## 3 **WebAsyncTask**

> Note that you can also set the default timeout value on a `DeferredResult`, a `ResponseBodyEmitter`, and an `SseEmitter`. For a `Callable`, you can use `WebAsyncTask` to provide a timeout value.
>
> 如果我们需要超时处理的回调或者错误处理的回调，我们可以使用`WebAsyncTask`代替Callable

### 案例

```java
@GetMapping("/webasync")
public WebAsyncTask<Object> testWebAsyncTask() {
    System.out.println(Thread.currentThread().getName() + "主线程开始");
    Callable<Object> callable = new Callable<Object>() {
        @Override
        public Object call() throws Exception {
            System.out.println(Thread.currentThread().getName() + " 子线程开始");
            TimeUnit.SECONDS.sleep(5);
            System.out.println(Thread.currentThread().getName() + " 子线程结束");
            Map<String, Object> map = new HashMap<>();
            map.put("name", "ken");
            map.put("age", "35");
            map.put("local", "中国");
            return map;
        }
    };
    System.out.println(Thread.currentThread().getName() + "主线程结束");
    WebAsyncTask<Object> webAsyncTask = new WebAsyncTask<>(6000, callable);
    // 注意：onCompletion表示完成，不管你是否超时、是否抛出异常，这个函数都会执行的
    webAsyncTask.onCompletion(() -> System.out.println("程序执行完成"));
    // 这两个返回的内容，最终都会放进response里面去===========
    webAsyncTask.onTimeout(() -> "程序[超时]的回调");
    // 备注：这个是Spring5新增的
    webAsyncTask.onError(() -> "程序[出现异常]的回调");

    return webAsyncTask;
}
```

### 源码

```java


package org.springframework.web.context.request.async;
import java.util.concurrent.Callable;

import org.springframework.beans.factory.BeanFactory;
import org.springframework.beans.factory.BeanFactoryAware;
import org.springframework.core.task.AsyncTaskExecutor;
import org.springframework.lang.Nullable;
import org.springframework.util.Assert;
import org.springframework.web.context.request.NativeWebRequest;

/**
 * Holder for a {@link Callable}, a timeout value, and a task executor.
 *
 * @author Rossen Stoyanchev
 * @author Juergen Hoeller
 * @since 3.2
 * @param <V> the value type
 */
public class WebAsyncTask<V> implements BeanFactoryAware {
    
	// 正常执行的函数（通过WebAsyncTask的构造函数可以传进来）
    private final Callable<V> callable;
    
	// 处理超时时间（ms），可通过构造函数指定，也可以不指定（不会有超时处理）
    private Long timeout;
    
	// 执行任务的执行器。可以构造函数设置进来，手动指定。
    private AsyncTaskExecutor executor;

    
	// 若设置了，会根据此名称去IoC容器里找这个Bean （和上面二选一）  
	// 若传了executorName,请务必调用set方法设置beanFactory
    private String executorName;

    private BeanFactory beanFactory;
	// 超时的回调
    private Callable<V> timeoutCallback;
    
	// 发生错误的回调
    private Callable<V> errorCallback;
    
	// 完成的回调（不管超时还是错误都会执行）
    private Runnable completionCallback;


    /**
	 * Create a {@code WebAsyncTask} wrapping the given {@link Callable}.
	 * @param callable the callable for concurrent handling
	 */
    public WebAsyncTask(Callable<V> callable) {
        Assert.notNull(callable, "Callable must not be null");
        this.callable = callable;
    }

    /**
	 * Create a {@code WebAsyncTask} with a timeout value and a {@link Callable}.
	 * @param timeout a timeout value in milliseconds
	 * @param callable the callable for concurrent handling
	 */
    public WebAsyncTask(long timeout, Callable<V> callable) {
        this(callable);
        this.timeout = timeout;
    }

    /**
	 * Create a {@code WebAsyncTask} with a timeout value, an executor name, and a {@link Callable}.
	 * @param timeout the timeout value in milliseconds; ignored if {@code null}
	 * @param executorName the name of an executor bean to use
	 * @param callable the callable for concurrent handling
	 */
    public WebAsyncTask(@Nullable Long timeout, String executorName, Callable<V> callable) {
        this(callable);
        Assert.notNull(executorName, "Executor name must not be null");
        this.executorName = executorName;
        this.timeout = timeout;
    }

    /**
	 * Create a {@code WebAsyncTask} with a timeout value, an executor instance, and a Callable.
	 * @param timeout the timeout value in milliseconds; ignored if {@code null}
	 * @param executor the executor to use
	 * @param callable the callable for concurrent handling
	 */
    public WebAsyncTask(@Nullable Long timeout, AsyncTaskExecutor executor, Callable<V> callable) {
        this(callable);
        Assert.notNull(executor, "Executor must not be null");
        this.executor = executor;
        this.timeout = timeout;
    }


    /**
	 * Return the {@link Callable} to use for concurrent handling (never {@code null}).
	 */
    public Callable<?> getCallable() {
        return this.callable;
    }

    /**
	 * Return the timeout value in milliseconds, or {@code null} if no timeout is set.
	 */
    @Nullable
    public Long getTimeout() {
        return this.timeout;
    }

    /**
	 * A {@link BeanFactory} to use for resolving an executor name.
	 * <p>This factory reference will automatically be set when
	 * {@code WebAsyncTask} is used within a Spring MVC controller.
	 */
    @Override
    public void setBeanFactory(BeanFactory beanFactory) {
        this.beanFactory = beanFactory;
    }

    /**
	 * Return the AsyncTaskExecutor to use for concurrent handling,
	 * or {@code null} if none specified.
	 */
    @Nullable
    public AsyncTaskExecutor getExecutor() {
        if (this.executor != null) {
            return this.executor;
        }
        else if (this.executorName != null) {
            Assert.state(this.beanFactory != null, "BeanFactory is required to look up an executor bean by name");
            return this.beanFactory.getBean(this.executorName, AsyncTaskExecutor.class);
        }
        else {
            return null;
        }
    }


    /**
	 * Register code to invoke when the async request times out.
	 * <p>This method is called from a container thread when an async request times
	 * out before the {@code Callable} has completed. The callback is executed in
	 * the same thread and therefore should return without blocking. It may return
	 * an alternative value to use, including an {@link Exception} or return
	 * {@link CallableProcessingInterceptor#RESULT_NONE RESULT_NONE}.
	 */
    public void onTimeout(Callable<V> callback) {
        this.timeoutCallback = callback;
    }

    /**
	 * Register code to invoke for an error during async request processing.
	 * <p>This method is called from a container thread when an error occurred
	 * while processing an async request before the {@code Callable} has
	 * completed. The callback is executed in the same thread and therefore
	 * should return without blocking. It may return an alternative value to
	 * use, including an {@link Exception} or return
	 * {@link CallableProcessingInterceptor#RESULT_NONE RESULT_NONE}.
	 * @since 5.0
	 */
    public void onError(Callable<V> callback) {
        this.errorCallback = callback;
    }

    /**
	 * Register code to invoke when the async request completes.
	 * <p>This method is called from a container thread when an async request
	 * completed for any reason, including timeout and network error.
	 */
    public void onCompletion(Runnable callback) {
        this.completionCallback = callback;
    }

    CallableProcessingInterceptor getInterceptor() {
        return new CallableProcessingInterceptor() {
            @Override
            public <T> Object handleTimeout(NativeWebRequest request, Callable<T> task) throws Exception {
                return (timeoutCallback != null ? timeoutCallback.call() : CallableProcessingInterceptor.RESULT_NONE);
            }
            @Override
            public <T> Object handleError(NativeWebRequest request, Callable<T> task, Throwable t) throws Exception {
                return (errorCallback != null ? errorCallback.call() : CallableProcessingInterceptor.RESULT_NONE);
            }
            @Override
            public <T> void afterCompletion(NativeWebRequest request, Callable<T> task) throws Exception {
                if (completionCallback != null) {
                    completionCallback.run();
                }
            }
        };
    }

}

```

> WebAsyncTask 的异步编程 API。相比于 @Async 注解，WebAsyncTask 提供更加健全的 超时处理 和 异常处理 支持。但是@Async也有更优秀的地方，就是他不仅仅能用于controller中~~~~（任意地方）



## **4 DeferredResult**

> `DeferredResult`使用方式与Callable类似，但在返回结果上不一样，**它返回的时候实际结果可能没有生成**，实际的结果可能会在**另外的线程**里面设置到`DeferredResult`中去。

### 官方Demo

> The controller can produce the return value asynchronously, from a different thread — for example, in response to an external event (JMS message), a scheduled task, or other event.

```java
@GetMapping("/quotes")
@ResponseBody
public DeferredResult<String> quotes() {
    DeferredResult<String> deferredResult = new DeferredResult<String>();
    // Save the deferredResult somewhere..
    return deferredResult;
}

// From some other thread...
deferredResult.setResult(result);
```

### 示例

> 我们第一个请求`/hello`，会先`deferredResult`存起来，然后前端页面是一直等待（转圈状态）的。知道我发第二个请求：`setHelloToAll`，所有的相关页面才会有响应~~

1. controller 返回一个`DeferredResult`，我们把它保存到内存里或者List里面（供后续访问）
2. Spring MVC调用`request.startAsync()`，开启异步处理
3. 与此同时将`DispatcherServlet`里的拦截器、Filter等等都马上退出主线程，**但是response仍然保持打开的状态**
4. 应用通过另外一个线程（可能是MQ消息、定时任务等）给`DeferredResult` set值。然后`Spring MVC`会把这个请求再次派发给servlet容器
5. `DispatcherServlet`再次被调用，然后处理后续的标准流程

```java
@GetMapping("/deferred")
public DeferredResult<String> helloGet() throws Exception {
    DeferredResult<String> deferredResult = new DeferredResult<>();

    //先存起来，等待触发
    deferredResultList.add(deferredResult);
    return deferredResult;
}

@PostMapping("/deferred")
public void helloSet() throws Exception {
    // 让所有hold住的请求给与响应
    deferredResultList.forEach(d -> d.setResult("say hello to all"));
}
```

### 源码

```java
public class DeferredResult<T> {

	private static final Object RESULT_NONE = new Object()

	
	// 超时时间（ms） 可以不配置
	@Nullable
	private final Long timeout;
	// 相当于超时的话的，传给回调函数的值
	private final Object timeoutResult;

	// 这三种回调也都是支持的
	private Runnable timeoutCallback;
	private Consumer<Throwable> errorCallback;
	private Runnable completionCallback;


	// 这个比较强大，就是能把我们结果再交给这个自定义的函数处理了 他是个@FunctionalInterface
	private DeferredResultHandler resultHandler;

	private volatile Object result = RESULT_NONE;
	private volatile boolean expired = false;


	// 判断这个DeferredResult是否已经被set过了（被set过的对象，就可以移除了嘛）
	// 如果expired表示已经过期了你还没set，也是返回false的
	// Spring4.0之后提供的
	public final boolean isSetOrExpired() {
		return (this.result != RESULT_NONE || this.expired);
	}

	// 没有isSetOrExpired 强大，建议使用上面那个
	public boolean hasResult() {
		return (this.result != RESULT_NONE);
	}

	// 还可以获得set进去的结果
	@Nullable
	public Object getResult() {
		Object resultToCheck = this.result;
		return (resultToCheck != RESULT_NONE ? resultToCheck : null);
	}


	public void onTimeout(Runnable callback) {
		this.timeoutCallback = callback;
	}
	public void onError(Consumer<Throwable> callback) {
		this.errorCallback = callback;
	}
	public void onCompletion(Runnable callback) {
		this.completionCallback = callback;
	}

	
	// 如果你的result还需要处理，可以这是一个resultHandler，会对你设置进去的结果进行处理
	public final void setResultHandler(DeferredResultHandler resultHandler) {
		Assert.notNull(resultHandler, "DeferredResultHandler is required");
		// Immediate expiration check outside of the result lock
		if (this.expired) {
			return;
		}
		Object resultToHandle;
		synchronized (this) {
			// Got the lock in the meantime: double-check expiration status
			if (this.expired) {
				return;
			}
			resultToHandle = this.result;
			if (resultToHandle == RESULT_NONE) {
				// No result yet: store handler for processing once it comes in
				this.resultHandler = resultHandler;
				return;
			}
		}
		try {
			resultHandler.handleResult(resultToHandle);
		} catch (Throwable ex) {
			logger.debug("Failed to handle existing result", ex);
		}
	}

	// 我们发现，这里调用是private方法setResultInternal，我们设置进来的结果result，会经过它的处理
	// 而它的处理逻辑也很简单，如果我们提供了resultHandler，它会把这个值进一步的交给我们的resultHandler处理
	// 若我们没有提供此resultHandler，那就保存下这个result即可
	public boolean setResult(T result) {
		return setResultInternal(result);
	}

	private boolean setResultInternal(Object result) {
		// Immediate expiration check outside of the result lock
		if (isSetOrExpired()) {
			return false;
		}
		DeferredResultHandler resultHandlerToUse;
		synchronized (this) {
			// Got the lock in the meantime: double-check expiration status
			if (isSetOrExpired()) {
				return false;
			}
			// At this point, we got a new result to process
			this.result = result;
			resultHandlerToUse = this.resultHandler;
			if (resultHandlerToUse == null) {
				this.resultHandler = null;
			}
		}
		resultHandlerToUse.handleResult(result);
		return true;
	}

	// 发生错误了，也可以设置一个值。这个result会被记下来，当作result
	// 注意这个和setResult的唯一区别，这里入参是Object类型，而setResult只能set规定的指定类型
	// 定义成Obj是有原因的：因为我们一般会把Exception等异常对象放进来。。。
	public boolean setErrorResult(Object result) {
		return setResultInternal(result);
	}

	// 拦截器 注意最终finally里面，都可能会调用我们的自己的处理器resultHandler(若存在的话)
	// afterCompletion不会调用resultHandler~~~~~~~~~~~~~
	final DeferredResultProcessingInterceptor getInterceptor() {
		return new DeferredResultProcessingInterceptor() {
			@Override
			public <S> boolean handleTimeout(NativeWebRequest request, DeferredResult<S> deferredResult) {
				boolean continueProcessing = true;
				try {
					if (timeoutCallback != null) {
						timeoutCallback.run();
					}
				} finally {
					if (timeoutResult != RESULT_NONE) {
						continueProcessing = false;
						try {
							setResultInternal(timeoutResult);
						} catch (Throwable ex) {
							logger.debug("Failed to handle timeout result", ex);
						}
					}
				}
				return continueProcessing;
			}
			@Override
			public <S> boolean handleError(NativeWebRequest request, DeferredResult<S> deferredResult, Throwable t) {
				try {
					if (errorCallback != null) {
						errorCallback.accept(t);
					}
				} finally {
					try {
						setResultInternal(t);
					} catch (Throwable ex) {
						logger.debug("Failed to handle error result", ex);
					}
				}
				return false;
			}
			@Override
			public <S> void afterCompletion(NativeWebRequest request, DeferredResult<S> deferredResult) {
				expired = true;
				if (completionCallback != null) {
					completionCallback.run();
				}
			}
		};
	}

	// 内部函数式接口 DeferredResultHandler
	@FunctionalInterface
	public interface DeferredResultHandler {
		void handleResult(Object result);
	}

}

```

> `DeferredResult`的超时处理，采用委托机制，也就是在实例`DeferredResult`时给予一个超时时长（毫秒），同时在`onTimeout`中委托（传入）一个新的处理线程（**我们可以认为是超时线程**）；当超时时间到来，`DeferredResult`启动超时线程，超时线程处理业务，封装返回数据，给`DeferredResult`赋值（正确返回的或错误返回的）

### 高级使用

#### 1 长轮询服务端推送消息（long polling）

在`WebSocket`协议之前(它是2011年发布的)，有三种实现双向通信的方式：**轮询（polling）**、**长轮询（long-polling）\**和\**iframe流（streaming）**。

- **轮询（polling）**：这个不解释了。优点是实现简单粗暴，后台处理简单。缺点也是大大的，耗流量、耗CPU
- **长轮询（long-polling）**：长轮询是对轮询的改进版。客户端发送HTTP给服务器之后，看有没有新消息，如果没有新消息，就一直等待（而不是一直去请求了）。当有新消息的时候，才会返回给客户端。 优点是对轮询做了优化，时效性也较好。**缺点是：保持连接会消耗资源; 服务器没有返回有效数据，程序超时**
- **iframe流（streaming）**：是在页面中插入一个`隐藏的iframe`，利用其src属性在服务器和客户端之间创建一条长连接，服务器向iframe传输数据（通常是HTML，内有负责插入信息的javascript），来实时更新页面。
- **WebSocket**：WebSocket协议是基于TCP的一种新的网络协议。它实现了浏览器与服务器全双工(full-duplex)通信——允许服务器主动发送信息给客户端。它将TCP的Socket（套接字）应用在了webpage上。 它的有点一大把：支持双向通信，实时性更强；可发送二进制文件；非常节省流量。 但也是有缺点的：`浏览器支持程度不一致`，不支持断开重连 .目前主流

`apollo配置中心`的实现原理，apollo的发布配置推送变更消息就是用`DeferredResult`实现的。它的大概实现步骤如下：

1. apollo客户端会像服务端发送`长轮询http请求`，超时时间60秒
2. 当超时后返回客户端一个304 httpstatus,表明配置没有变更，客户端`继续这个步骤重复发起请求`
3. 当有发布配置的时候，服务端会调用`DeferredResult.setResult`返回200状态码。客户端收到响应结果后，**会发起请求获取变更后的配置信息**（注意这里是另外一个请求）

#### 2 Demo

```java
@Configuration
@EnableWebMvc
public class AppConfig implements WebMvcConfigurer {

    @Override
    public void configureAsyncSupport(AsyncSupportConfigurer configurer) {
        // 超时时间设置为60s
        configurer.setDefaultTimeout(TimeUnit.SECONDS.toMillis(60));
    }
}

```

**服务端简单代码模拟如下：**

```java
@Slf4j
@RestController
public class ApolloController {

    // 值为List，因为监视同一个名称空间的长轮询可能有N个（毕竟可能有多个客户端用同一份配置嘛）
    private Map<String, List<DeferredResult<String>>> watchRequests = new ConcurrentHashMap<>();

    @GetMapping(value = "/all/watchrequests")
    public Object getWatchRequests() {
        return watchRequests;
    }

    // 模拟长轮询：apollo客户端来监听配置文件的变更~  可以指定namespace 监视指定的NameSpace
    @GetMapping(value = "/watch/{namespace}")
    public DeferredResult<String> watch(@PathVariable("namespace") String namespace) {
        log.info("Request received,namespace is" + namespace + ",当前时间：" + System.currentTimeMillis());

        DeferredResult<String> deferredResult = new DeferredResult<>();

        //当deferredResult完成时（不论是超时还是异常还是正常完成），都应该移除watchRequests中相应的watch key
        deferredResult.onCompletion(() -> {
            log.info("onCompletion，移除对namespace：" + namespace + "的监视~");
            List<DeferredResult<String>> list = watchRequests.get(namespace);
            list.remove(deferredResult);
            if (list.isEmpty()) {
                watchRequests.remove(namespace);
            }
        });

        List<DeferredResult<String>> list = watchRequests.computeIfAbsent(namespace, (k) -> new ArrayList<>());
        list.add(deferredResult);
        return deferredResult;


    }

    //模拟发布namespace配置：修改配置
    @GetMapping(value = "/publish/{namespace}")
    public void publishConfig(@PathVariable("namespace") String namespace) {
        //do Something for update config

        if (watchRequests.containsKey(namespace)) {
            List<DeferredResult<String>> deferredResults = watchRequests.get(namespace);

            //通知所有watch这个namespace变更的长轮训配置变更结果
            for (DeferredResult<String> deferredResult : deferredResults) {
                deferredResult.setResult(namespace + " changed，时间为" + System.currentTimeMillis());
            }
        }

    }
}

```

apollo处理超时时候会抛出一个异常`AsyncRequestTimeoutException`，因此我们全局处理一下就成：

```java
@Slf4j
@ControllerAdvice
class GlobalControllerExceptionHandler {

    @ResponseStatus(HttpStatus.NOT_MODIFIED)//返回304状态码  效果同HttpServletResponse#sendError(int) 但这样更优雅
    @ResponseBody
    @ExceptionHandler(AsyncRequestTimeoutException.class) //捕获特定异常
    public void handleAsyncRequestTimeoutException(AsyncRequestTimeoutException e) {
        System.out.println("handleAsyncRequestTimeoutException");
    }
}
```

**用`Ajax`模拟Client端的伪代码如下：**

```java
//长轮询：一直去监听指定namespace的配置文件
function watchConfig(){
    $.ajax({
        url:"http://localhost:8080/demo_war/watch/classroomconfig",
        method:"get",
        success:function(response,status){
            if(status == 304){
                watchConfig(); //超时，没有更改，那就继续去监听
            }else if(status == 200){
                getNewConfig(); //监听到更改后，立马去获取最新的配置文件内容回来做事
                ...

                    watchConfig(); // 昨晚事后又去监听着
            }
        }

    });
}

// 调用去监听获取配置文件的函数
watchConfig();
```

## 5 ResponseBodyEmitter与SseEmitter

> `Callback`和`DeferredResult`用于设置单个结果，如果有多个结果需要set返回给客户端时，可以使用`SseEmitter以及ResponseBodyEmitter`，each object is written with a compatible `HttpMessageConverter`。返回值可以直接写他们本身，也可以放在`ResponseEntity`里面
>
> 它俩都是Spring4.2之后提供的类。由`ResponseBodyEmitterReturnValueHandler`负责处理。 这个和Spring5提供的webFlux技术已经很像
> Emitter：发射器

它们的使用方式几乎同：`DeferredResult`

```java
@GetMapping("/events")
public ResponseBodyEmitter handle() {
    ResponseBodyEmitter emitter = new ResponseBodyEmitter();
    // Save the emitter somewhere..
    return emitter;
}

// In some other thread
emitter.send("Hello once");

// and again later on
emitter.send("Hello again");

// and done at some point
emitter.complete();
```

`SseEmitter`是`ResponseBodyEmitter`的子类,它提供`Server-Sent Events（Sse）`.服务器事件发送是”HTTP Streaming”的另一个变种技术.只是从服务器发送的事件按照`W3C Server-Sent Events`规范来的（推荐使用） 它的使用方式上，完全同上

```java
@GetMapping(path="/events", produces=MediaType.TEXT_EVENT_STREAM_VALUE)
public SseEmitter handle() {
    SseEmitter emitter = new SseEmitter();
    // Save the emitter somewhere..
    return emitter;
}

// In some other thread
emitter.send("Hello once");

// and again later on
emitter.send("Hello again");

// and done at some point
emitter.complete();
```





`Server-Sent Events`这个规范能够来用于它们的预期使用目的：就是从server发送events到clients（服务器推）.在Spring MVC中可以很容易的实现.仅仅需要返回一个`SseEmitter`类型的值.

> 向这种场景在在线游戏、在线协作、金融领域等等都有很好的应用。当然，如果你对稳定性什么的要求都非常高，官方也推荐最好是使用`WebSocket`来实现~

`ResponseBodyEmitter`允许通过`HttpMessageConverter`把发送的events写到对象到response中.这可能是最常见的情况。**例如写JSON**

## 6 StreamingResponseBody 

```java
@Override
public ResponseEntity<StreamingResponseBody> downLoad(String fileName) throws IOException {

    MediaType mediaType = MediaTypeFactory
        .getMediaType(fileName)
        .orElse(MediaType.APPLICATION_OCTET_STREAM);
    HttpHeaders headers = new HttpHeaders();
    headers.setContentType(mediaType);
    ContentDisposition disposition = ContentDisposition.builder("attachment")
        .filename(new String(fileName.getBytes("UTF-8"), "ISO-8859-1"))
        .build();
    headers.setContentDisposition(disposition);
    return new ResponseEntity<StreamingResponseBody>(outputStream -> {
        try (BufferedOutputStream stream = new BufferedOutputStream(outputStream);
             BufferedInputStream inputStream = new BufferedInputStream(ftpUtils.downFile(fileName));) {
            StreamUtils.copy(inputStream, stream);
        } finally {
            this.ftpUtils.ftpClient.completePendingCommand();
        }

    }, headers, HttpStatus.OK);

}
```

## 8 异步优化

Spring内部默认不使用线程池处理的（通过源码分析后面我们是能看到的）,为了提高处理的效率，我们可以自己优化，建议自己在配置里注入一个线程池供给使用，参考如下：

```java
	// 提供一个mvc里专用的线程池。。。  这是全局的方式~~~~
    @Bean
    public ThreadPoolTaskExecutor mvcTaskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setQueueCapacity(100);
        executor.setMaxPoolSize(25);
        return executor;
    }

// 最优解决方案不是像上面一样配置通用的，而是配置一个单独的专用的，如下~~~~
@Configuration
@EnableWebMvc
public class WebMvcConfig extends WebMvcConfigurerAdapter {

	// 配置异步支持~~~~
    @Override
    public void configureAsyncSupport(AsyncSupportConfigurer configurer) {
    	// 设置一个用于异步执行的执行器~~~AsyncTaskExecutor
        configurer.setTaskExecutor(mvcTaskExecutor());
        configurer.setDefaultTimeout(60000L);
    }
}

```



# 2 Filter与HandlerInterceptor

### Filter

```java
// 注意，这里必须开启异步支持asyncSupported = true，否则报错：Async support must be enabled on a servlet and for all filters involved in async request processing
@WebFilter(urlPatterns = "/*", asyncSupported = true)
public class HelloFilter extends OncePerRequestFilter {

    @Override
    protected void initFilterBean() throws ServletException {
        System.out.println("Filter初始化...");
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        System.out.println(Thread.currentThread().getName() + "--->" + request.getRequestURI());
        filterChain.doFilter(request, response);
    }

}

```

### **HandlerInterceptor**

```java
public class HelloInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println(Thread.currentThread().getName() + "---preHandle-->" + request.getRequestURI());
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println(Thread.currentThread().getName() + "---postHandle-->" + request.getRequestURI());
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println(Thread.currentThread().getName() + "---postHandle-->" + request.getRequestURI());
    }
}

// 注册拦截器
@Configuration
@EnableWebMvc
public class AppConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
    	// /**拦截所有请求
        registry.addInterceptor(new HelloInterceptor()).addPathPatterns("/**");
    }
}

```

> 从上面可以看出，如果我们就是普通的Spring MVC的拦截器，preHandler会执行两次，这也符合我们上面分析的处理步骤。**所以我们在书写preHandler的时候，一定要特别的注意，要让preHandler即使执行多次，也不要受到影响（幂等）**

### 异步拦截器

> Spring MVC给提供了异步拦截器，能让我们更深入的参与进去异步request的生命周期里面去。其中最为常用的为：

#### `AsyncHandlerInterceptor`

```java
public class AsyncHelloInterceptor implements AsyncHandlerInterceptor {

    // 这是Spring3.2提供的方法，专门拦截异步请求的方式
    @Override
    public void afterConcurrentHandlingStarted(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println(Thread.currentThread().getName() + "---afterConcurrentHandlingStarted-->" + request.getRequestURI());
    }

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println(Thread.currentThread().getName() + "---preHandle-->" + request.getRequestURI());
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println(Thread.currentThread().getName() + "---postHandle-->" + request.getRequestURI());
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println(Thread.currentThread().getName() + "---afterCompletion-->" + request.getRequestURI());
    }
}

```

> `AsyncHandlerInterceptor`提供了一个`afterConcurrentHandlingStarted()`方法, 这个方法会在`Controller`方法异步执行时开始执行, 而`Interceptor的postHandle`方法则是需要等到`Controller`的异步执行完才能执行
>
> **（比如我们用`DeferredResult`的话，`afterConcurrentHandlingStarted`是在return的之后执行，而`postHandle()`是执行`.setResult()`之后执行）**
>
> 需要说明的是：如果我们不是异步请求，`afterConcurrentHandlingStarted`是不会执行的。所以我们可以把它当做加强版的`HandlerInterceptor`来用。平时我们若要使用拦截器，建议使用它。（Spring5，JDK8以后，很多的`xxxAdapter`都没啥用了，直接implements接口就成~）

同样可以注册`CallableProcessingInterceptor`或者一个`DeferredResultProcessingInterceptor`用于更深度的集成异步request的生命周期

```java
@Override
public void configureAsyncSupport(AsyncSupportConfigurer configurer) {
    // 注册异步的拦截器、默认的超时时间、任务处理器TaskExecutor等等
    //configurer.registerCallableInterceptors();
    //configurer.registerDeferredResultInterceptors();
    //configurer.setDefaultTimeout();
    //configurer.setTaskExecutor();
}

```

# 3  @Async注解

> 在开发过程中，我们会遇到很多使用线程池的业务场景，例如异步短信通知、异步记录操作日志。大多数使用线程池的场景，就是会将一些可以进行异步操作的业务放在线程池中去完成。
>
> 例如在生成订单的时候给用户发送短信，生成订单的结果不应该被发送短信的成功与否所左右，也就是说生成订单这个主操作是不依赖于发送短信这个操作，所以我们就可以把发送短信这个操作置为异步操作。

Spring中用`@Async`注解标记的方法，称为异步方法，它会在调用方的当前线程之外的独立的线程中执行，其实就相当于我们自己`new Thread(()-> System.out.println("hello world !"))`这样在另一个线程中去执行相应的业务逻辑

```java
// @Async 若把注解放在类上或者接口上，那么他所有的方法都会异步执行了~~~~（包括私有方法）
public interface HelloService {
    Object hello();
}

@Service
public class HelloServiceImpl implements HelloService {

    @Async // 注意此处加上了此注解
    @Override
    public Object hello() {
        System.out.println("当前线程：" + Thread.currentThread().getName());
        return "service hello";
    }
}

```

然后只需要在配置里，开启对异步的支持即可：

```java
@Configuration
@EnableAsync // 开启异步注解的支持
public class RootConfig {

}
```

#### @Async注解使用细节

1. `@Async`注解一般用在方法上，如果用在类上，那么这个类**所有的方法**都是异步执行的；
2. `@Async`可以放在任何方法上，哪怕你是`private`的（**若是同类调用**，请务必注意注解失效的情况~~~）
3. 所使用的`@Async`注解方法的类对象应该是Spring容器管理的bean对象
4. `@Async`可以放在接口处（或者接口方法上）。但是只有使用的是JDK的动态代理时才有效，CGLIB会失效。因此建议：`统一写在实现类的方法上`
5. 需要注解`@EnableAsync`来开启异步注解的支持
6. 若你希望得到`异步调用的返回值`，请你的返回值用`Futrue`变量包装起来
7. 

# 4 总结



> `DeferredResult`需要**自己用线程**来处理结果`setResult`，而`Callable`的话不需要我们来维护一个结果处理线程。
> 总体来说，`Callable`的话更为简单，同样的也是因为简单，灵活性不够；
> 相对地，`DeferredResult`更为复杂一些，**但是又极大的灵活性**，所以能实现非常多个性化的、复杂的功能，可以设计高级应用。

**有些较常见的场景， `Callable`也并不能解决，比如说：我们访问A接口，A接口调用三方的服务，服务`回调（注意此处指的回调，不是返回值）`B接口，这种情况就没办法使用Callable了，这个时候可以使用`DeferredResult`**

使用原则：基本上在可以用`Callable`的时候，直接用`Callable`；而遇到`Callable`没法解决的场景的时候，可以尝试使用`DeferredResult`。

> 在Reactive编程模型越来越流行的今天，多一点对异步编程模型（Spring MVC异步模式）的了解，可以更容易去接触Spring5带来的新特性—响应式编程。
> 同时，异步编程是我们高效利用系统资源，提高系统吞吐量，编写高性能应用的必备技能。

# 5 参阅

[【小家Spring】高性能关键技术之---体验Spring MVC的异步模式（Callable、WebAsyncTask、DeferredResult） 基础使用篇]: https://blog.csdn.net/f641385712/article/details/88692534?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522160778625119725222472012%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&amp;request_id=160778625119725222472012&amp;biz_id=0&amp;utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_v2~rank_v29-2-88692534.pc_v2_rank_blog_default&amp;utm_term=%E9%AB%98%E6%80%A7%E8%83%BD&amp;spm=1018.2118.3001.4450
[【小家Spring】高性能关键技术之---体验Spring MVC的异步模式（ResponseBodyEmitter、SseEmitter、StreamingResponseBody） 高级使用篇]: https://blog.csdn.net/f641385712/article/details/88710676?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522160778625119725222472012%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&amp;request_id=160778625119725222472012&amp;biz_id=0&amp;utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_v2~rank_v29-1-88710676.pc_v2_rank_blog_default&amp;utm_term=%E9%AB%98%E6%80%A7%E8%83%BD&amp;spm=1018.2118.3001.4450
[Spring文档]: https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-http-streaming



[Spring文档]: https://docs.spring.io/spring-framework/docs/current/reference/html/overview.html#overview
[]: https://fangshixiang.blog.csdn.net/article/details/89430276

