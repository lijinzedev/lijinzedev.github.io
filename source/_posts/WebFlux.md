---
title: WebFlux
top: false
cover: false
toc: true
mathjax: true
categories:
  - SpringMvc
tags:
  - SpringMvc
date: 2020-12-15 22:59:57
password:
summary:
---

# WebFlux

GitHub Demo

# 1 Servlet异步

**1 为什么要使用异步Servlet,Servlet阻塞了什么**

> 增加吞吐量，tomcat容器的servlet线程

**2 异步Servlet如何进行工作**

> 

# 2 WebFlux基础

> reactor=jdk8 stream +jdk9 reactive stream

## 1 Mono

> 0-1 个元素

```java
@GetMapping("/1")
public Mono<String> testMono() {
    return Mono.just("Hello Mono");
}
```

## 2 Flux

> 0-N个元素

```java
// 定义订阅者
Subscriber<Integer> subscriber = new Subscriber<Integer>() {

    private Subscription subscription;

    @Override
    public void onSubscribe(Subscription subscription) {
        // 保存订阅关系
        this.subscription = subscription;
        this.subscription.request(1);
    }

    @Override
    public void onNext(Integer integer) {
        System.out.println("接受到的数据为" + integer);
        // 继续请求数据
        this.subscription.request(1);
        // 或者告诉发布者不再接受数据
        //                this.subscription.cancel();
    }

    @Override
    public void onError(Throwable throwable) {
        throwable.printStackTrace();
        // 或者告诉发布者不再接受数据
        //                this.subscription.cancel();
    }

    @Override
    public void onComplete() {
        System.out.printf("数据处理完成");
    }
};
String[] strs = {"1", "2", "3", "4"};
Flux.fromArray(strs).map(Integer::parseInt).subscribe(subscriber);
```

## 3 样例

```java
@GetMapping(value = "/2", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<String> testFlux() {
	Flux<String> flux = Flux.fromStream(IntStream.range(1, 5).mapToObj(value -> {
	try {
        TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
        return "Hello world " + value;
    }));
    return flux;
}
```

# 3 SSE(server-send events)

> H5的SSE,用于服务器向前台推送数据的场景,聊天室等

## 1 服务端

```java
protected void doGet(HttpServletRequest request,HttpServletResponse response){
    response.setContentType("text/event-stream");
    response.setCharacterEncoding("utf-8");
    for (int i = 0; i < ; i++) {
        // 指定事件标志
        response.getWriter().write("event:me\n");
        // 格式 data + 数据 +两个回车
		response.getWriter().write("data:"+i+"\n\n");
        response.getWriter().flush();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    
}
```

## 2 客户端

```javascript
<script type="text/javascript">
    // 初始化参数为URL
    // 依赖H5
    var sse = new EventSource("SSE");
	sse.onmessage=function(e){
        console.log("message:",e.data,e);
    };
	sse.addEvnetListener("me",function(e){
         console.log("event:",e.data);
        if(e.data==3){
            sse.close();
        }
    })
</script>   
```

# 4  CRUD示例

### 1 Repository 

```java
/**
 * @author: Curiosity
 * @Date: 2020/12/19 17:15
 * @Description:
 */
@Repository
public interface UserRepository extends ReactiveMongoRepository<User, String> {
    /**
     *
     * @param start
     * @param end
     * @return
     */
    Flux<User> findByAgeBetween(int start, int end);
}

```



### 2 Controller

```java
@RestController
@RequestMapping("/user")
@RequiredArgsConstructor
public class UserController {
    private final UserRepository userRepository;

    @GetMapping("/")
    public Flux<User> findAll() {
        return userRepository.findAll();
    }

    @GetMapping(value = "/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<User> streamFindAll() {
        return userRepository.findAll();
    }

    @PostMapping(value = "/", consumes = MediaType.APPLICATION_JSON_VALUE)
    public Mono<User> createUser(@RequestBody User user) {
        // Spring Data Jpa 里面 新增和修改都是 save id为空是新增,id不为空是修改
        // 设置id为空
        user.setId(null);
        return userRepository.save(user);
    }

    @DeleteMapping(value = "/{id}")
    public Mono<ResponseEntity<Void>> deleteUserById(@PathVariable("id") String id) {
        // 不能判断是否存在
        //        userRepository.deleteById(id);
        // 当你要操作数据,并返回一个Mono,这个时候使用flatMap,如果不操作数据而是转换数据这个时候使用map
        return this.userRepository.findById(id).flatMap(user -> userRepository.delete(user)).then(Mono.just(new ResponseEntity<Void>(HttpStatus.OK)))
                .defaultIfEmpty(new ResponseEntity<Void>(HttpStatus.NOT_FOUND));
    }

    @PutMapping(value = "/{id}")
    public Mono<ResponseEntity<User>> updateUser(@PathVariable("id") String id, @RequestBody User user) {
        return this.userRepository.findById(id).flatMap(u -> {
            u.setAge(user.getAge());
            u.setName(user.getName());
            return this.userRepository.save(u);
        }).map(u -> new ResponseEntity<>(u, HttpStatus.OK)).defaultIfEmpty(new ResponseEntity<>(HttpStatus.NOT_FOUND));
    }
    
    @GetMapping("/age/{start}/{end}")
    public Flux<User> findByAge(@PathVariable("start") Integer start, @PathVariable("end") Integer end) {
        return this.userRepository.findByAgeBetween(start, end);
    }
    @GetMapping(value = "/stream/age/{start}/{end}",produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<User> streamFindByAge(@PathVariable("start") Integer start, @PathVariable("end") Integer end) {
        return this.userRepository.findByAgeBetween(start, end);
    }
}

```

# 5 RouterFunction 模式

> 1. ServerRequest  <-> HttpServletRequest
> 2. ServerResponse  <-> HttpServletResponse
>
> 开发过程
>
> HandlerFunction（输入ServerRequest返回ServerResponse）
>
> 1. RouterFunction（请求URL和HandlerFunction对应起来）
>
> 2. 包装成HttpHandler
>
> 3. Server处理

## 1 样例

### 1 Handler

```java

/**
 * @author: Curiosity
 * @Date: 2020/12/19 21:14
 * @Description:
 */
@Component
@RequiredArgsConstructor
public class UserHandler {
    private final UserRepository userRepository;

    /**
     * 查询所有的用户
     *
     * @param serverRequest
     * @return
     */
    public Mono<ServerResponse> getAllUsers(ServerRequest serverRequest) {
        return ok().contentType(MediaType.APPLICATION_JSON).
                body(this.userRepository.findAll(), User.class);
    }

    /**
     * 创建用户
     *
     * @param serverRequest
     * @return
     */
    public Mono<ServerResponse> createUser(ServerRequest serverRequest) {
        Mono<User> userMono = serverRequest.bodyToMono(User.class);
        // 注意这里我们不用对Mono进行消费,并且任何时候都不能进行消费,需要Spring框架进行消费
        return userMono.flatMap(u -> {
            // 校验代码
            CheckUtil.checkName(u.getName());
            return ok().contentType(MediaType.APPLICATION_JSON).
                    body(this.userRepository.save(u), User.class);
        });

    }

    /**
     * 删除用户
     */
    public Mono<ServerResponse> deleteUserByUserId(ServerRequest serverRequest) {
        String id = serverRequest.pathVariable("id");
        return this.userRepository.findById(id).flatMap(u ->
                this.userRepository.delete(u).then(ok().build()).switchIfEmpty(notFound().build())
        );
    }
}

```

### 2 Router

```java
/**
 * @author: Curiosity
 * @Date: 2020/12/19 21:35
 * @Description:
 */
@Configuration
public class AllRouters {
    @Bean
    RouterFunction<ServerResponse> userRouter(UserHandler userHandler) {
        return nest(
                // 相当于RequestMapping的注释
                path("/user/router"),
                // 相当于GetMapping等
                route(GET("/"), userHandler::getAllUsers)).
                andRoute(RequestPredicates.POST("/"), userHandler::createUser).
                andRoute(RequestPredicates.DELETE("/"), userHandler::deleteUserByUserId);
    }
}

```



## 2 参数校验

### 1 Handler

```java
/**
 * @author: Curiosity
 * @Date: 2020/12/19 22:02
 * @Description:
 */
@Component
// 因为有很多默认实现的Handler所以需要调高优先级,让我们的Handler生效
@Order(-2)
public class ExceptionHandler implements WebExceptionHandler {
    @Override
    public Mono<Void> handle(ServerWebExchange serverWebExchange, Throwable throwable) {
        ServerHttpResponse response = serverWebExchange.getResponse();
        // 设置响应头
        response.setStatusCode(HttpStatus.BAD_REQUEST);
        //  设置享应类型
        response.getHeaders().setContentType(MediaType.APPLICATION_JSON);
        String str = toStr(throwable);
        DataBuffer db = response.bufferFactory().wrap(str.getBytes());

        return response.writeWith(Mono.just(db));

    }

    private String toStr(Throwable throwable) {
        // 已知异常
        if (throwable instanceof RuntimeException) {

            // 未知异常答应堆栈信息
        } else {
            throwable.printStackTrace();
        }
        return "发生了异常";
    }
}

```

# 6 Client设计

## 1 设计思想

> 大佬的思想:
>
> 1. 程序 = 数据结构+ 算法
>
> 2. 设计最重要的是解耦，而实现解耦最关键的是设计自己的数据机构 + 抽象接口

![图](image-20201219222924014.png)

## 2 demo

