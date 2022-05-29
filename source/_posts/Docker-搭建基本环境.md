---
title: Docker 搭建基本环境
top: false
cover: false
toc: true
mathjax: true
categories:
  - 运维
  - docker
tags:
  - 运维
  - docker
date: 2021-01-05 16:51:36
password:
summary:
---

# 1 Docker 安装Mysql 

```bash
# 下载镜像
docker pull mysql:5.7
# 运行
docker run -d -p 3306:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql_ruoyi mysql:5.7
# 安装8.0
docker pull mysql:8.0.20
docker run -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=123456  -d mysql:8.0.20
docker cp  mysql:/etc/mysql /home/docker/mysql8.0.20
docker rm -f  mysql
docker run \
-p 3306:3306 \
--name mysql \
--privileged=true \
--restart unless-stopped \
-v /home/docker/mysql8.0.20/mysql:/etc/mysql \
-v /home/docker/mysql8.0.20/logs:/logs \
-v /home/docker/mysql8.0.20/data:/var/lib/mysql \
-v /etc/localtime:/etc/localtime \
-e MYSQL_ROOT_PASSWORD=123456 \
-d mysql:8.0.20
```

# 2 Docker安装Nacos

```bash
# 获取
docker pull nacos/nacos-server

# 启动容器
docker run --env MODE=standalone --name nacos -d -p 8848:8848 nacos/nacos-server
# 
docker cp nacos:/home/nacos /home/win10/
# 单机模式运行 
docker run --name nacos -d -p 8848:8848 --privileged=true --restart=always -e JVM_XMS=256m -e JVM_XMX=512m -e MODE=standalone -e PREFER_HOST_MODE=hostname -v /home/nacos/logs:/home/nacos/logs -v /home/nacos/conf/:/home/nacos/conf/ -v /home/nacos/init.d:/home/nacos/init.d  nacos/nacos-server

docker run -it -e MODE=standalone \
-e SPRING_DATASOURCE_PLATFORM=mysql \
-e MYSQL_MASTER_SERVICE_HOST=172.17.0.1 \
-e MYSQL_MASTER_SERVICE_DB_NAME=nacos_config \
-e MYSQL_MASTER_SERVICE_PORT=3306 \
-e MYSQL_MASTER_SERVICE_USER=root \
-e MYSQL_MASTER_SERVICE_PASSWORD=123456 \
-v /home/nacos/logs:/home/nacos/logs \
--restart=always \
--name nacos -p 8848:8848 nacos/nacos-server

# 新版本
docker run -it -e MODE=standalone \
-e SPRING_DATASOURCE_PLATFORM=mysql \
-e MYSQL_SERVICE_HOST=192.168.173.103 \
-e MYSQL_SERVICE_DB_NAME=nacos_conifg \
-e MYSQL_SERVICE_PORT=3307 \
-e MYSQL_SERVICE_USER=root \
-e MYSQL_SERVICE_PASSWORD=123456 \
-v /home/nacos/logs:/home/nacos/logs \
--restart=always \
--name nacos -p 8848:8848 nacos/nacos-server

# 在浏览器访问 
http://localhost:8848/nacos/index.html
默认登录账号 nacos/nacos
```

# 3 Docker安装Redis

```bash
# 下载镜像
docker pull redis 
# 创建目录
mkdir -p /home/docker/redis/conf
mkdir -p /home/docker/redis/data

# 新增配置文件
cd /home/redis/conf
vim redis.conf
-----------------
# 开启远程访问
#bind 127.0.0.1 
protected-mode no
# 开启数据持久化
appendonly yes
# 设置访问密码
requirepass 123456 
------------------
# 创建容器并启动
docker run --name redis -p 6379:6379 -v /home/redis/data:/data -v /home/redis/conf/redis.conf:/etc/redis/redis.conf -d redis redis-server /etc/redis/redis.conf 
```

# 4 Docker部署Java项目

```dockerfile
FROM java:8
MAINTAINER Curiosity
ADD ./target/ruoyi-modules-system-2.4.0.jar ruoyi-modules-system.jar
EXPOSE 9201
ENTRYPOINT ["java","-jar","ruoyi-modules-system.jar"]
```





# 5 Docker 部署Vue项目

## 1 Nginx 配置文件

```nginx
user  root root;
worker_processes  4;
worker_rlimit_nofile 65535; 
worker_cpu_affinity 0001 0010 0100 1000;
error_log  logs/error.log warn;
pid logs/nginx.pid; 

events {
    use epoll;
    worker_connections 65535;
    accept_mutex off;
    multi_accept off;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  access  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  logs/access.log  access;

    sendfile        on;
    tcp_nopush      on;
    tcp_nodelay      on;
    keepalive_timeout  65;
    server_tokens off;
    fastcgi_intercept_errors on;
    charset utf-8;
    server_names_hash_bucket_size 128; 
    client_header_buffer_size 4k; 
    large_client_header_buffers 8 128k;
    client_max_body_size 300m;

    gzip on;
    gzip_min_length 1k;
    gzip_buffers 4 16k;
    gzip_http_version 1.1;  
    gzip_comp_level 5;
    gzip_types gzip_types text/plain application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png; 
    gzip_vary on;
    gzip_disable "MSIE [1-6]\.";
    
    server {
        listen       80;
        server_name  yzypc;
        location / {
            root   /dist/;
            index  index.html;
            client_max_body_size 100m; 
            client_body_buffer_size 512k;
            proxy_send_timeout 300;
            proxy_read_timeout 300;
            proxy_connect_timeout 300;          
            proxy_buffer_size 64k;
            proxy_buffers 16 64k; 
            proxy_busy_buffers_size 64k;
            proxy_temp_file_write_size 64k;         
            proxy_set_header Host $host:$server_port;  
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header REMOTE-HOST $remote_addr;          
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
            add_header Access-Control-Allow-Origin *;
            add_header Access-Control-Allow-Headers "Origin, X-Requested-With, Content-Type, Accept";
            add_header Access-Control-Allow-Methods "GET, POST, OPTIONS";
        } 
        location /prod-api {
            proxy_pass http://42.193.118.181:8080/;
            client_max_body_size 100m; 
            client_body_buffer_size 512k;
            proxy_send_timeout 300;
            proxy_read_timeout 300;
            proxy_connect_timeout 300;          
            proxy_buffer_size 64k;
            proxy_buffers 16 64k; 
            proxy_busy_buffers_size 64k;
            proxy_temp_file_write_size 64k;         
            proxy_set_header Host $host:$server_port;  
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header REMOTE-HOST $remote_addr;          
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
            add_header Access-Control-Allow-Origin *;
            add_header Access-Control-Allow-Headers "Origin, X-Requested-With, Content-Type, Accept";
            add_header Access-Control-Allow-Methods "GET, POST, OPTIONS";
        }
        
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
        
        location /favicon.ico {  
             root html;  
        }       
        
    }

}

```

## 2 Dockerfile

```dockerfile
FROM nginx
MAINTAINER Curiosity
RUN mkdir -p /etc/nginx/logs
RUN mkdir -p /dist
COPY ./nginx.conf /etc/nginx/nginx.conf
COPY ./dist /dist
EXPOSE 80
```

