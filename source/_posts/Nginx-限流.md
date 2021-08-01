---
title: Nginx 限流
top: false
cover: false
toc: true
mathjax: true
categories:
  - nginx
tags:
  - nginx
date: 2021-07-31 23:50:10
password:
summary:
---

# Nginx 限流篇

实验环境：

* 树莓派4b ubuntu server
* nginx arm64v8-1.21.1

## 一、前言

随着业务的扩散，系统并发越来越高时,有三样利器用来保护系统，分别是缓存、降级和限流。

**缓存:**缓存是现在系统中必不可少的模块,并且已经成为了高并发高性能架构的一个关键组件，缓存的目的是提升系统访问速度和增大系统处理容量。

**降级:**这个在天猫双11的时候非常常见，降级是当服务出现问题或者影响到核心流程时，需要暂时屏蔽掉，待高峰或者问题解决后再打开。

**限流:**限流的目的是通过对并发访问/请求进行限速，或者对一个时间窗口内的请求进行限速来保护系统，一旦达到限制速率则可以拒绝服务、排队或等待、降级等处理

## 二、Docker环境搭建

```bash
# 创建文件夹
mkdir -p /docker/nginx/{html,logs}
cd /docker/nginx
# 拷贝文件
docker run --name tmp-nginx-container -d arm64v8/nginx
docker cp tmp-nginx-container:/etc/nginx/nginx.conf $(pwd)
docker rm -f tmp-nginx-container

# 启动nginx
docker run -d -p 8080:80 --name nginx -v $(pwd)/html:/usr/share/nginx/html -v $(pwd)/nginx.conf:/etc/nginx/nginx.conf -v $(pwd)/logs:/var/log/nginx arm64v8/nginx
# 测试 创建index.html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>nginx</title>
</head>
<body>
    <h1>hello world!</h1>
</body>
</html>
# 访问
curl localhost:8080
```

## 三、Nginx如何限流

`Nginx`的”流量限制”使用漏桶算法(*leaky bucket algorithm*)，该算法在通讯和分组交换计算机网络中广泛使用，用以处理带宽有限时的突发情况。就好比，一个桶口在倒水，桶底在漏水的水桶。如果桶口倒水的速率大于桶底的漏水速率，桶里面的水将会溢出；同样，在请求处理方面，水代表来自客户端的请求，水桶代表根据”先进先出调度算法”(FIFO)等待被处理的请求队列，桶底漏出的水代表离开缓冲区被服务器处理的请求，桶口溢出的水代表被丢弃和不被处理的请求。

### 3.1 配置基本的限流

> Nginx自身有的请求限制模块[ngx_http_limit_req_module](http://nginx.org/en/docs/http/ngx_http_limit_req_module.html)、流量限制模块[ngx_stream_limit_conn_module](http://nginx.org/en/docs/http/ngx_http_limit_conn_module.html)基于令牌桶算法，可以方便的控制令牌速率，自定义调节限流，实现基本的限流控制。
>
> 对于提供下载的网站，肯定是要进行流量控制的，例如软件下载站、视频服务等。
> 它也可以减少一些爬虫程序或者DDOS的攻击。

**1 添加limit_zone和limit_req_zone**

```nginx
limit_zone one  $binary_remote_addr  20m;
limit_req_zone  $binary_remote_addr  zone=req_one:20m rate=12r/s;
```

**2 添加limit_conn 和limit_req**

这个变量可以在`http`, `server`, `location`使用 如果限制nginx上的所有服务，可以添加到http里面 （如果你需要限制部分服务，可在nginx/conf/domains里面选择相应的server或者location添加上便可）

```nginx
limit_conn one 10;
limit_req zone=req_one burst=120;
```

```nginx
# 是针对每个变量(这里指IP，即$binary_remote_addr)定义一个存储session状态的容器。这个示例中定义了一个20m的容器，按照32bytes/session，可以处理640000个session。
limit_zone，
# 与limit_zone类似。rate是请求频率. 每秒允许12个请求。
limit_req_zone 
# 表示一个IP能发起10个并发连接数
limit_conn  one 10 
# 与limit_req_zone对应。burst表示缓存住的请求数。
limit_req:
```

#### 3.1.1 limit_req_zone  速率控制

```nginx
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=10r/s;

server {
	location /login/ {
		limit_req zone=mylimit;
		proxy_pass http://my_upstream;
	}
}
```

##### 3.1.1.1 定义

用来限制单位时间内的请求数，限制单一的IP地址的请求的处理速率，即速率限制,采用的漏桶算法 "leaky bucket"。

`limit_req_zone`指令通常在HTTP块中定义，使其可在多个上下文中使用，它需要以下三个参数：

- **Key** - 定义应用限制的请求特性。示例中的Nginx变量`$binary_remote_addr`，保存客户端IP地址的二进制形式。这意味着，我们可以将每个不同的IP地址限制到，通过第三个参数设置的请求速率。(使用该变量是因为比字符串形式的客户端IP地址`$remote_addr`，占用更少的空间)
- **Zone** - 定义用于存储每个IP地址状态以及被限制请求URL访问频率的共享内存区域。保存在内存共享区域的信息，意味着可以在Nginx的worker进程之间共享。定义分为两个部分：通过`zone=keyword`标识区域的名字，以及冒号后面跟区域大小。16000个IP地址的状态信息，大约需要1MB，所以示例中区域可以存储160000个IP地址。
- **Rate** - 定义最大请求速率。在示例中，速率不能超过每秒10个请求。Nginx实际上以毫秒的粒度来跟踪请求，所以速率限制相当于每100毫秒1个请求。因为不允许”突发情况”(见下一章节)，这意味着在前一个请求100毫秒内到达的请求将被拒绝。、

> 当Nginx需要添加新条目时存储空间不足，将会删除旧条目。如果释放的空间仍不够容纳新记录，Nginx将会返回 **503状态码**(Service Temporarily Unavailable)。另外，为了防止内存被耗尽，Nginx每次创建新条目时，最多删除两条60秒内未使用的条目。

[`limit_req_zone`](http://nginx.org/en/docs/http/ngx_http_limit_req_module.html#limit_req_zone)指令定义了流量限制相关的参数，而[`limit_req`](http://nginx.org/en/docs/http/ngx_http_limit_req_module.html#limit_req)指令在出现的上下文中启用流量限制(示例中，对于”/login/”的所有请求)。

`limit_req_zone`指令设置流量限制和共享内存区域的参数，但实际上并不限制请求速率。所以需要通过添加`limit_req`指令，将流量限制应用在特定的`location`或者`server`块。在上面示例中，我们对`/login/`请求进行流量限制。

现在每个IP地址被限制为每秒只能请求10次`/login/`，更准确地说，在前一个请求的100毫秒内不能请求该URL。

##### 3.1.1.2 简单使用

在http中添加：`limit_req_zone $binary_remote_addr zone=mylimit:10m rate=10r/s;`

* 第一个参数：$binary_remote_addr 表示通过remote_addr这个标识来做限制，“binary_”的目的是缩写内存占用量，是限制同一客户端ip地址。
* 第二个参数：zone=one:10m表示生成一个大小为10M，名字为one的内存区域，用来存储访问的频次信息。
* 第三个参数：rate=1r/s表示允许相同标识的客户端的访问频次，这里限制的是每秒10次，还可以有比如30r/m的。

限流速度为每秒10次请求，如果有10次请求同时到达一个空闲的nginx，他们都能得到执行吗？

![空桶](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210801172809.png)

漏桶漏出请求是匀速的。10r/s是怎样匀速的呢？每100ms漏出一个请求。

在这样的配置下，桶是空的，所有不能实时漏出的请求，都会被拒绝掉。

所以如果10次请求同时到达，那么只有一个请求能够得到执行，其它的，都会被拒绝。

##### 3.1.1.3 处理突发

如果我们在100毫秒内接收到2个请求，怎么办？对于第二个请求，Nginx将给客户端返回状态码503。这可能并不是我们想要的结果，因为应用本质上趋向于突发性。相反地，我们希望缓冲任何超额的请求，然后及时地处理它们。我们更新下配置，在`limit_req`中使用`burst`参数：

```nginx
location /login/ {
	limit_req zone=mylimit burst=12;
	proxy_pass http://my_upstream;
}
```

* 第一个参数：zone=one 设置使用哪个配置区域来做限制，与上面limit_req_zone 里的name对应。

* 第二个参数：burst=12，重点说明一下这个配置，burst是爆发的意思，这个配置的意思是设置一个大小为5的缓冲区当有大量请求（爆发）过来时，超过了访问频次限制的请求可以先放到这个缓冲区内。例如，如果从一个给定IP地址发送13个请求，Nginx会立即将第一个请求发送到上游服务器群，然后将余下12个请求放在队列中。然后每100毫秒转发一个排队的请求，只有当传入请求使队列中排队的请求数超过12时，Nginx才会向客户端返回503。

  ![Burst](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210801185744.png)

逻辑上叫漏桶，实现起来是FIFO队列，把得不到执行的请求暂时缓存起来。

这样漏出的速度仍然是100ms一个请求，但并发而来，暂时得不到执行的请求，可以先缓存起来。只有当队列满了的时候，才会拒绝接受新请求。

这样漏桶在限流的同时，也起到了削峰填谷的作用。

在这样的配置下，如果有10次请求同时到达，它们会依次执行，每100ms执行1个。

虽然得到执行了，但因为排队执行，延迟大大增加，在很多场景下仍然是不能接受的。

##### 3.1.1.4 无延迟的排队

配置`burst`参数将会使通讯更流畅，但是可能会不太实用，因为该配置会使站点看起来很慢。在上面的示例中，队列中的第20个包需要等待2秒才能被转发，此时返回给客户端的响应可能不再有用。要解决这个情况，可以在`burst`参数后添加`nodelay`参数：

```nginx
# 存储空间为10m 每秒钟只允许有10个请求经过
limit_req_zone $binary_remote_addr zone=ip_limit:10m rate=10r/s;
server {
    location /login/ {
        limit_req zone=ip_limit burst=12 nodelay;
        proxy_pass http://login_upstream;
    }
}

```

第三个参数：nodelay，如果设置，超过访问频次而且缓冲区也满了的时候就会直接返回503，如果没有设置，则所有请求会等待排队。

也就是说使用`nodelay`参数，Nginx仍将根据`burst`参数分配队列中的位置，并应用已配置的速率限制，而不是清理队列中等待转发的请求。相反地，当一个请求到达“太早”时，只要在队列中能分配位置，Nginx将立即转发这个请求。将队列中的该位置标记为”taken”(占据)，并且不会被释放以供另一个请求使用，直到一段时间后才会被释放(在这个示例中是，100毫秒后)。

假设如前所述，队列中有12个空位，从给定的IP地址发出的13个请求同时到达。Nginx会立即转发这个13个请求，并且标记队列中占据的12个位置，然后每100毫秒释放一个位置。如果是15个请求同时到达，Nginx将会立即转发其中的13个请求，标记队列中占据的12个位置，并且返回503状态码来拒绝剩下的2个请求。

现在假设，第一组请求被转发后101毫秒，另20个请求同时到达。队列中只会有一个位置被释放，所以Nginx转发一个请求并返回503状态码来拒绝其他19个请求。如果在20个新请求到达之前已经过去了501毫秒，5个位置被释放，所以Nginx立即转发5个请求并拒绝另外15个。

效果相当于每秒10个请求的“流量限制”。如果希望不限制两个请求间允许间隔的情况下实施“流量限制”，`nodelay`参数是很实用的。

**注意：** 对于大部分部署，建议使用`burst`和`nodelay`参数来配置`limit_req`指令。

![NoDelay](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210801185740.png)

要么立刻执行，要么被拒绝，请求不会因为限流而增加延迟了。

因为请求从桶里漏出来还是匀速的，桶的空间又是固定的，最终平均下来，还是每秒执行了5次请求，限流的目的还是达到了。

但这样也有缺点，限流是限了，但是限得不那么匀速。以上面的配置举例，如果有12个请求同时到达，那么这12个请求都能够立刻执行，然后后面的请求只能匀速进桶，100ms执行1个。如果有一段时间没有请求，桶空了，那么又可能出现并发的12个请求一起执行。

##### 3.1.1.5 delay

大部分情况下，这种限流不匀速，不算是大问题。不过nginx也提供了一个参数才控制并发执行也就是nodelay的请求的数量。

```nginx
limit_req_zone $binary_remote_addr zone=ip_limit:10m rate=10r/s;

server {
    location /login/ {
        limit_req zone=ip_limit burst=12 delay=4;
        proxy_pass http://login_upstream;
    }
}
```

- delay=4 从桶内第5个请求开始delay

![DelayNum](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210801185907.png)

这样通过控制delay参数的值，可以调整允许并发执行的请求的数量，使得请求变的均匀起来，在有些耗资源的服务上控制这个数量，还是有必要的。



#### 3.1.2 limit_req_conn 并发控制

用来限制同一时间连接数，即并发限制。

ngx_http_limit_conn_module这个模块用来限制单个IP的请求数。并非所有的连接都被计数。只有在服务器处理了请求并且已经读取了整个请求头时，连接才被计数。

**语法**

> limit_conn_zone $variable zone=name:size;

**参数说明:**

- $variable: 定义的键，要限流的维度,[$binary_remote_addr:同一客户端IP，$server_name:限制同一server最大并发数]
- size: 定义各个键共享内存空间大小
- name: zone名称

```nginx
# 按IP配置内存区域zone的大小为10m:
limit_conn_zone $binary_remote_addr zone=perip_conn:10m;

# 按server配置一个连接 zone
limit_conn_zone $server_name zone=perserver_conn:100m;
```

##### 3.1.2.1 limit_conn

```nginx
 # 限制同一个IP同一时间来源的连接数:10个
 limit_conn perip_conn 10;
 # 限制同一个虚拟服务器同一时的总连接数
 limit_conn perserver_conn 2000;
```

##### 3.1.2.2 limit_rate 限速

```nginx
# 从下载指定的文件大小(1M)之后开始限速
limit_rate_after 1m;
# 限制请求资源
limit_rate 1k;
```

##### 3.1.2.3 配置限制固定连接数

```nginx
#根据IP地址来限制，存储内存大小10M
limit_conn_zone $binary_remote_addr zone=addr:1m;
#测试
location /user/ {
     # 表示 同一个地址只允许连接2次。
     limit_conn addr 2;
     proxy_pass http://172.16.0.235:18083;
}
```

##### 3.1.2.4 限制每个客户端IP与服务器的连接数，同时限制与虚拟服务器的连接总数

```nginx
limit_conn_zone $binary_remote_addr zone=perip:10m;
limit_conn_zone $server_name zone=perserver:10m; 

#测试
location /user/ {
     #limit_conn addr 2;
     # 单个客户端ip与服务器的连接数．
     limit_conn perip 10;
     # 限制与服务器的总连接数
     limit_conn perserver 100; 
     proxy_pass http://172.16.0.235:18083;
}
```

## 四、高级配置示例



通过将基本的“流量限制”与其他Nginx功能配合使用，我们可以实现更细粒度的流量限制。

### 4.1 白名单

下面这个例子将展示，如何对任何不在白名单内的请求强制执行“流量限制”：

```nginx
geo $limit {
	default 		1;
	10.0.0.0/8 		0;
	192.168.0.0/64 	0;
}

map $limit $limit_key {
	0 "";
	1 $binary_remote_addr;
}

limit_req_zone $limit_key zone=req_zone:10m rate=5r/s;

server {
	location / {
		limit_req zone=req_zone burst=10 nodelay;

		# ...
	}
}
```

这个例子同时使用了[`geo`](http://nginx.org/en/docs/http/ngx_http_geo_module.html#geo)和[`map`](http://nginx.org/en/docs/http/ngx_http_map_module.html#map)指令。`geo`块将给在白名单中的IP地址对应的`$limit`变量分配一个值0，给其它不在白名单中的分配一个值1。然后我们使用一个映射将这些值转为key，如下：

- 如果`$limit`变量的值是0，`$limit_key`变量将被赋值为空字符串
- 如果`$limit`变量的值是1，`$limit_key`变量将被赋值为客户端二进制形式的IP地址

两个指令配合使用，白名单内IP地址的`$limit_key`变量被赋值为空字符串，不在白名单内的被赋值为客户端的IP地址。当`limit_req_zone`后的第一个参数是空字符串时，不会应用“流量限制”，所以白名单内的IP地址(10.0.0.0/8和192.168.0.0/24 网段内)不会被限制。其它所有IP地址都会被限制到每秒5个请求。

`limit_req`指令将限制应用到**/**的location块，允许在配置的限制上最多超过10个数据包的突发，并且不会延迟转发。

### 4.2 location包含多`limit_req`指令

我们可以在一个location块中配置多个`limit_req`指令。符合给定请求的所有限制都被应用时，意味着将采用最严格的那个限制。例如，多个指令都制定了延迟，将采用最长的那个延迟。同样，请求受部分指令影响被拒绝，即使其他指令允许通过也无济于事。

扩展前面将“流量限制”应用到白名单内IP地址的例子：

```ngxin
http {
	# ...

	limit_req_zone $limit_key zone=req_zone:10m rate=5r/s;
	limit_req_zone $binary_remote_addr zone=req_zone_wl:10m rate=15r/s;

	server {
		# ...
		location / {
			limit_req zone=req_zone burst=10 nodelay;
			limit_req zone=req_zone_wl burst=20 nodelay;
			# ...
		}
	}
}
```

白名单内的IP地址不会匹配到第一个“流量限制”，而是会匹配到第二个`req_zone_wl`，并且被限制到每秒15个请求。不在白名单内的IP地址两个限制能匹配到，所以应用限制更强的那个：每秒5个请求。

## 五、配置相关功能



### 5.1 日志记录

默认情况下，Nginx会在日志中记录由于流量限制而延迟或丢弃的请求，如下所示：

```
2015/06/13 04:20:00 [error] 120315#0: *32086 limiting requests, excess: 1.000 by zone "mylimit", client: 192.168.1.2, server: nginx.com, request: "GET / HTTP/1.0", host: "nginx.com"
```

日志条目中包含的字段：

- *limiting requests* - 表明日志条目记录的是被“流量限制”请求
- *excess* - 每毫秒超过对应“流量限制”配置的请求数量
- *zone* - 定义实施“流量限制”的区域
- *client* - 发起请求的客户端IP地址
- *server* - 服务器IP地址或主机名
- *request* - 客户端发起的实际HTTP请求
- *host* - HTTP报头中host的值

默认情况下，Nginx以`error`级别来记录被拒绝的请求，如上面示例中的`[error]`所示(Ngin以较低级别记录延时请求，一般是`info`级别)。如要更改Nginx的日志记录级别，需要使用[`limit_req_log_level`](http://nginx.org/en/docs/http/ngx_http_limit_req_module.html#limit_req_log_level)指令。这里，我们将被拒绝请求的日志记录级别设置为`warn`：

```
location /login/ {
	limit_req zone=mylimit burst=20 nodelay;
	limit_req_log_level warn;
	
	proxy_pass http://my_upstream;
} 
```

### 5.2 发送到客户端的错误代码

一般情况下，客户端超过配置的流量限制时，Nginx响应状态码为**503(Service Temporarily Unavailable)**。可以使用[`limit_req_status`](http://nginx.org/en/docs/http/ngx_http_limit_req_module.html#limit_req_status)指令来设置为其它状态码(例如下面的**444**状态码):

```
location /login/ {
	limit_req zone=mylimit burst=20 nodelay;
	limit_req_status 444;
}
```

### 指定`location`拒绝所有请求

如果你想拒绝某个指定URL地址的所有请求，而不是仅仅对其限速，只需要在[`location`](http://nginx.org/en/docs/http/ngx_http_core_module.html#location)块中配置[`deny`](http://nginx.org/en/docs/http/ngx_http_access_module.html#deny) **all**指令：

```
location /foo.php {
	deny all;
}
```

## 六、总结

前文已经涵盖了Nginx和Nginx Plus提供的“流量限制”的很多功能，包括为HTTP请求的不同loation设置请求速率，给“流量限制”配置`burst`和`nodelay`参数。还涵盖了针对客户端IP地址的白名单和黑名单应用不同“流量限制”的高级配置，阐述了如何去日志记录被拒绝和延时的请求。

## 七、拓展

动态控制，参考https://github.com/openresty/lua-resty-limit-traffic

## 八、参考文档

[图解Nginx限流配置](https://zhaox.github.io/nginx/2019/09/05/explain-nginx-rate-limit)

