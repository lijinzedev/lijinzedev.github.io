---
title: Nginx配置HTTPS
top: false
cover: false
toc: true
mathjax: true
categories:
  - 中间件
  - nginx
tags:
  - 中间件
  - nginx
date: 2021-04-02 16:32:52
password:
summary:
---

# nginx配置https

## 一、介绍

起初是在部署系统时，用扫描漏洞工具扫描系统，发现网站访问不安全，要求使用https安全认证访问web，而 `nginx` 支持 `https` 技术，所以就在 `nginx` 配置了个 `https` ；

本教程是在 `Centos7` 上配置，其他版本的 `linux` 改一下对应的命令即可。供参考。

配置之前，是 `http` 的连接方式。

![](https://gitee.com/ycwgit/mypic/raw/master/img/20210331111105.png)

配置完成，访问浏览器后，网站前面会出现红色的叉，这是因为在网络服务器上找不到对应的证书厂商，不妨碍使用。

![](https://gitee.com/ycwgit/mypic/raw/master/img/20210331111503.png)

不用在意这里的ip地址不一样，只做效果展示。

## 二、准备

> 准备两个安装包**nginx**、**openssl**

### 方法1、手动上传（离线上传）

`nginx` 官方下载地址：[下载地址](http://nginx.org/en/download.html)

`openssl` 官方下载地址：[下载地址](https://www.openssl.org/source/old/)

下载完，自行通过 `ftp工具`传上去。我这里放在`/usr/local`下面

### 方法2、在线下载

```shell
# 下载最新nginx-1.18.0
curl -O http://nginx.org/download/nginx-1.18.0.tar.gz
或者
wget http://nginx.org/download/nginx-1.18.0.tar.gz

# 下载最新openssl-1.1.1j
https://www.openssl.org/source/old/1.1.1/openssl-1.1.1j.tar.gz
```



## 三、安装nginx，配置ssl

### 3.1、首次安装nginx + ssl

进入`/usr/local`目录

#### 3.1.1 解压

```shell
tar zxvf nginx-1.18.0.tar.gz
tar zxvf openssl-1.1.1j.tar.gz
```

#### 3.1.2 配置nginx参数

```shell
# 进入nginx源码目录下，配置nginx
[root@hbtc local]# cd /usr/local/nginx-1.18.0

./configure --prefix=/usr/local/nginx-1.18.0 --with-http_stub_status_module --with-http_ssl_module --with-openssl=/usr/local/openssl-1.1.1j
```

#### 3.1.3 开始编译

```shell
[root@hbtc nginx-1.18.0]# make

# 此时当前目录会出现 objs 文件夹
```

#### 3.1.4 安装nginx

```shell
[root@hbtc nginx-1.18.0]# make install 

# 如果之前已经安装nginx的，这里就不再make install，否则会覆盖掉之前的安装和配置
```

#### 3.1.5 备份原先的启动文件

```shell
[root@hbtc nginx-1.18.0]# cp ./sbin/nginx ./sbin/nginx.back
```

#### 3.1.6 查看模块

```shell
[root@hbtc nginx-1.18.0]# ./sbin/nginx -V
nginx version: nginx/1.18.0
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC) 
built with OpenSSL 1.1.1j  16 Feb 2021
TLS SNI support enabled
configure arguments: --prefix=/usr/local/nginx-1.18.0 --with-http_stub_status_module --with-http_ssl_module --with-openssl=/usr/local/openssl-1.1.1j
```

有 `configure arguments: --with-http_ssl_module` 说明ssl模块已安装

### 3.2、已安装nginx，但没有安装ssl模块

#### 3.2.1 查看已配置的参数

进入之前已安装 `/usr/local/nginx/` 目录，使用命令：`./sbin/nginx -V ` 查看已配置的参数

- 查看 nginx 是否安装 `http_ssl_module` 模块。

```shell
[root@hbtc nginx]# ./sbin/nginx -V
configure arguments: --prefix=/usr/local/nginx  --add-module=/root/soft/ngx_devel_kit-0.3.0 --add-module=/root/soft/lua-nginx-module-0.10.9rc7
或者是
configure arguments: --with-pcre=../pcre-8.42 --with-zlib=../zlib-1.2.11 --with-openssl=../openssl-fips-2.0.16
```

* 如果出现 `configure arguments: --with-http_ssl_module` , 则已安装（下面的步骤可以跳过，进入 `nginx.conf` 配置）。

* 如果没有，记下这里的配置参数，在下面的配置中需要用到。

#### 3.2.2 备份原先的启动文件

```shell
[root@hbtc nginx]# cp ./sbin/nginx ./sbin/nginx.back
```

#### 3.2.3 配置nginx，需要把之前的配置参数加上（若不加会影响原有的功能）

```shell
./configure  --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module  --with-openssl=/usr/local/openssl-1.1.1j  --add-module=/root/soft/ngx_devel_kit-0.3.0 --add-module=/root/soft/lua-nginx-module-0.10.9rc7 
```

#### 3.2.4 开始编译

**注意因为之前已经安装nginx的，这里就不再make install，否则会覆盖掉之前的安装和配置**

```shell
[root@hbtc nginx]# make
```

**此时当前目录会出现 objs 文件夹，进入到objs下，拷贝nginx到安装目录下**

#### 3.2.5 拷贝新的 nginx 启动文件

```shell
# 用新的 nginx 文件覆盖当前的 nginx 文件

[root@hbtc nginx]# cp ./objs/nginx ./sbin
```

#### 3.2.6 再次查看安装的模块

```shell
./sbin/nginx -V

# 有 configure arguments: --with-http_ssl_module 说明ssl模块已安装
```

**至此，nginx支持ssl模块安装完毕！**

**加载ssl模块后，会在nginx.conf加上配置文件HTTPS SERVER 后面的ssl信息**

## 四、安装证书

nginx支持https协议需要服务器证书，此证书使用openssl命令生成（确保openssl命令可用）

证书生成步骤如下： 

### 4.1 新建证书存放目录

```shell
# 进入到/usr/local/nginx-1.18.0/conf/下，新建cert目录，并进入
[root@hbtc conf]# mkdir cert
[root@hbtc conf]# cd cert

```

### 4.2 生成证书

* **生成证书和密钥 -des3: CBC模式的DES加密，以下示例生成一个1024位的RSA私钥**

```shell
[root@hbtc cert]# openssl genrsa -des3 -out server.key 1024
```

出现以下信息：

```shell
Generating RSA private key, 1024 bit long modulus
...............++++++
.................................++++++
e is 65537 (0x10001)
Enter pass phrase for server.key:（此处随意输入证书密码，比如123456）
Verifying - Enter pass phrase for server.key:（重复输入一次）
```

* **创建服务器证书的申请文件 server.csr**

```shell
openssl req -new -key server.key -out server.csr
```

(注：此步骤生成证书，需要输入国家/地区/公司/个人相关信息，不需要真实，内容差不多就行，可参考下面的加粗部分)

```shell
[root@hbtc nginx-1.18.0]# openssl req -new -key server.key -out server.csr
Enter pass phrase for server.key:（输入上面的密码）
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN	# 国家缩写
State or Province Name (full name) []:BeiJing	# 省份
Locality Name (eg, city) [Default City]:BeiJing	# 市
Organization Name (eg, company) [Default Company Ltd]:hbtc	# 公司名
Organizational Unit Name (eg, section) []:					# 组织名，可以不填
Common Name (eg, your name or your server's hostname) []:	# 公共名，可以不填
Email Address []:											# 邮箱地址，可以不填

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:									# 加强的密码，可以不填
An optional company name []:								# 可以不填
```

* **备份文件，跳过证书验证密码**

```shell
[root@hbtc cert]# cp server.key server.key.org
[root@hbtc cert]# openssl rsa -in server.key.org -out server.key
```

* **生成证书， 证书有效天数(如果输入9999表示永久) 签名，开启双向认证**

```shell
[root@hbtc cert]# openssl x509 -req -days 9999 -in server.csr -signkey server.key -out server.crt
Signature ok
subject=/C=CN/ST=BeiJing/L=BeiJing/O=hbtc
Getting Private key
```

 **到此，证书创建完毕。**

## 五、配置nginx.conf

注意ssl证书文件一定得先准备好。

`nginx.conf` 配置文件的相关配置介绍，可以参考nginx的[官方配置资料](http://nginx.org/en/docs/http/configuring_https_servers.html#single_http_https_server)

进入到 `/usr/local/nginx-1.18.0/conf` ，修改 `nginx.conf` ，简单配置如下：

````nginx
# 单个server可以80和433共存，但80的还是http
#  HTTPS server
#
server {
	listen 443 ssl;
	server_name localhost;
	#ssl on;		#nginx1.15版本之前需要加，之后的不用加
	ssl_certificate /usr/local/nginx-1.18.0/conf/cert/server.crt;
	ssl_certificate_key /usr/local/nginx-1.18.0/conf/cert/server.key;
	ssl_protocols TLSv1 TLSv1.1 TLSv1.2; 
	ssl_prefer_server_ciphers on;

	#其它的一些配置
	
}
````

**本人项目完整配置**

```nginx
[root@hbtc nginx-1.18.0]# cat ./conf/nginx.conf 

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;		# 替换成你的域名
        charset UTF8;
		return  301 https://$server_name$request_uri;	//新版本重定向语句
        # rewrite ^(.*)$ https://${server_name}$1 permanent;	//旧版本重定向语句

        #access_log  logs/host.access.log  main;

        #location / {
        #    root   html;
        #    index  index.html index.htm;
        #}
        
    }

    # HTTPS server
    #

    server {
        listen       443 ssl;
        server_name  localhost;		# 替换成你的域名
		# ssl on; nginx1.15版本之前需要加，之后的不用加
        ssl_certificate      cert/server.crt;	#放置服务器证书的目录，替换成你的pem文件名称
        ssl_certificate_key  cert/server.key;	#放置服务器私钥的目录，替换成你的key文件名称

        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;

        ssl_ciphers  HIGH:!aNULL:!MD5;			# 使用此加密套件
        ssl_prefer_server_ciphers  on;

        location / {
            root   /home/testServer/dist/;   	# 页面地址
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

        location /api/ {
            proxy_pass http://172.16.7.14:8090/;	# 代理地址
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
    }

}
```

## 六、nginx相关命令

```shell
# 启动nginx
./sbin/nginx -c ./conf/nginx.conf

# 关闭nginx进程
nginx -s stop

# 重新加载配置文件
nginx –s reload

# 检查配置文件是否有误
nginx –t
```

## 七、常见问题

* **启动时nginx:[emerg]unknown directive ssl错误**

**原因是nginx缺少SSL模块**，需要重新将SSL模块添加进去，然后再启动nginx：

1. **查看是否有模块`./sbin/nginx -V`**

2. **在解压目录，执行命令；重新编译nginx**

````shell
[root@hbtc nginx-1.18.0]# cd /usr/local/src/nginx-1.10.0

[root@hbtc nginx-1.18.0]# ./configure --prefix=/usr/local/nginx-1.18.0 --with-http_stub_status_module --with-http_ssl_module --with-openssl=/usr/local/openssl-1.1.1j

[root@hbtc nginx-1.18.0]# make
````

这一步千万不能 `make install` ；不然会把之前已经安装的nginx 覆盖掉

3. **之后会看在当前目录生成objs文件，执行`./objs/nginx -V`**

```shell
nginx version: nginx/1.18.0
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC) 
built with OpenSSL 1.1.1j  16 Feb 2021
TLS SNI support enabled
configure arguments: --prefix=/usr/local/nginx-1.18.0 --with-http_stub_status_module --with-http_ssl_module --with-openssl=/usr/local/openssl-1.1.1j
```

发现 `TLS SNI support enabled` 这我们可以放心用了，这可以实现一个ip多个站点。

但是 `nginx -v` 这时候还是老版本的nginx，我们需要先备份

```shell
mv /usr/local/nginx-1.18.0/sbin/nginx /usr/local/nginx-1.18.0/sbin/nginx.old
```

然后，将 `objs` 目录下的 `nginx` 文件复制到 `/usr/local/nginx-1.18.0/sbin/` 下覆盖，然后重新启动即可。

测试下

```shell
/usr/local/nginx/sbin/nginx -t
```

ok，执行更新

```shell
make upgrade
```

4. **再次查看nginx的模块，看下是否把需要的模块编译进去了**

```shell
[root@hbtc nginx-1.18.0]# ./sbin/nginx -V
nginx version: nginx/1.18.0
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC) 
built with OpenSSL 1.1.1j  16 Feb 2021
TLS SNI support enabled
configure arguments: --prefix=/usr/local/nginx-1.18.0 --with-http_stub_status_module --with-http_ssl_module --with-openssl=/usr/local/openssl-1.1.1j
```

到此就成功了升级了 nginx 并且添加了 TLS SNI support 。

5. **重新启动nginx；**

```shell
./sbin/nginx  -s reload
```

* **nginx: [alert] could not open error log file: open() "/usr/local/nginx-1.18.0/logs/error.log" failed (2: No such file or directory)**

  **原因是缺少这个logs目录**，新建这个目录即可 `mkdir logs`

* 暂未实现http自动跳转到https，推测是因为 `server_name` 没有配置成域名的原因

* 双向验证的配置，参考链接

https://blog.csdn.net/witmind/article/details/78456660?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control&dist_request_id=&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control

https://blog.csdn.net/YYBDESHIJIE/article/details/109238535?utm_medium=distribute.pc_relevant.none-task-blog-baidujs_title-1&spm=1001.2101.3001.4242