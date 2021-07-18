---
ctitle: 树莓派安装Ubuntu
top: false
cover: false
toc: true
mathjax: true
categories:
  - 树莓派
tags:
  - 树莓派
date: 2021-07-13 21:37:45
password:
summary:

---

# 一 前言

> 由于树莓派的原生系统arm架构在使用docker的时候处处都不方便这边换了Ubuntu系统搭建docker环境

```ruby
# 看到armv7就是arm 看到armv8或者aarch64就是arm64
ubuntu@ubuntu:~$ uname -a
Linux ubuntu 5.4.0-1028-raspi #31-Ubuntu SMP PREEMPT Wed Jan 20 11:30:45 UTC 2021 aarch64 aarch64 aarch64 GNU/Linux
```

# 二 系统安装

## 1 下载地址

[ubuntu server的官方引导下载地址](https://ubuntu.com/download/raspberry-pi/thank-you?version=20.04&architecture=arm64+raspi)

[清华大学的镜像库下载地址](https://mirrors.tuna.tsinghua.edu.cn/ubuntu-cdimage/ubuntu/releases/20.04/release/)

![image-20210713214929791](https://raw.githubusercontent.com/lijinzedev/picture/main/img20210713214931.png)

我[下载](https://mirrors.tuna.tsinghua.edu.cn/ubuntu-cdimage/ubuntu/releases/20.04/release/ubuntu-20.04.2-preinstalled-server-arm64+raspi.img.xz)的是上图这个，arm64版本，因为我在的树莓派上装docker 很多镜像没有对应的arm/v7版本，所以打算装一个arm64版本的ubuntu server。

## 2 安装教程

[参阅](https://ubuntu.com/tutorials/create-an-ubuntu-image-for-a-raspberry-pi-on-windows#2-on-your-windows-machine)

# 三 开机配置

## 1.1 开启ssh

- 为了能够在*没插显示器*的情况下，能够进入ssh远程配置
- 在boot分区中，新建空白文件`ssh`，注意没有任何后缀

## 1.2 连接网络

* 因为没插显示器，所以需要开机前配置好网络

* 此处提供两种方法

## 1.3 在boot分区中，找到network-config文件，在里面更改成你的wifi信息

```ruby
wifis:
	wlan0:
		dhcp4: true
		optional: true
		access-points:
			wifi名称:
				password: “密码”
```

##   1.4 或者直接插网线

* 插路由器LAN口

## 1.5 插电开机

## 1.6找到设备的ip

​    这里不作过多赘述，一般在路由器的管理界面可以找到，设备名是ubuntu

## 1.6用ssh登陆设备，（用户名和密码都是ubuntu）

```ruby
ssh ubuntu@[你的ip]
```
## 1.7 登陆成功后会要求你重新设置密码

## 1.8 开启ssh的方法

```
# sudo apt search openssh-server
# sudo apt install opessh-client openssh-server
# sudo dpkg-reconfigure openssh-server
# sudo service ssh restart
# sudo service ssh status
# sudo systemctl enable ssh
```



# 四 环境搭建

## [1 设置静态IP地址](https://www.myfreax.com/how-to-configure-static-ip-address-on-ubuntu-20-04/)

## 2 换源

```bash
 sudo sed -i 's/ports.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
```

## 3 安装Docker

```bash
#使用 apt-get 进行安装
# step 1: 安装必要的一些系统工具
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
# step 2: 安装GPG证书
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
# Step 3: 写入软件源信息
sudo add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
# Step 4: 更新并安装Docker-CE
sudo apt-get -y update
sudo apt-get -y install docker-ce
 
# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
# apt-cache madison docker-ce
#   docker-ce | 17.03.1~ce-0~ubuntu-xenial | https://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
#   docker-ce | 17.03.0~ce-0~ubuntu-xenial | https://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
# Step 2: 安装指定版本的Docker-CE: (VERSION例如上面的17.03.1~ce-0~ubuntu-xenial)
# sudo apt-get -y install docker-ce=[VERSION]
```

**PS: 安装失败请看这里**

>  ubuntu安装docker-ce提示Package 'docker-ce' has no installation candidate解决的办法

```bash
# 添加docker源
sudo echo "deb https://download.docker.com/linux/ubuntu zesty edge" > /etc/apt/sources.list.d/docker.list
# 支持解析https
sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
# 设置存储库位置
sudo add-apt-repository"deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
# 安装
sudo apt-get install -y docker-ce
```

### 3.1 用户与用户组

> 不用sudo 执行Docker命令
>
> 默认情况下，只能以root用户或由docker组中的用户运行docker命令，而docker组是在Docker安装过程中自动创建的。 为了不使用sudo前缀，需要加入doccker组：

```bash
# 查看所有用户和用户组：
cat /etc/passwd
cat /etc/group
# 为该用户添加sudo权限
sudo usermod -a -G adm wyx
sudo usermod -a -G sudo wyx
# 检查自己在哪些组的命令是
id -nG
# 添加到docker组
sudo usermod -aG docker ${USER}
```

### 3.2 安装Docker-compose

[Docker-compose安装](https://github.com/docker/compose/issues/6831)

[linuxserver/docker-compose](https://hub.docker.com/r/linuxserver/docker-compose)

## 4 安装Redis

```shell
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
docker run --name redis -p 6379:6379 -v /home/docker/redis/data:/data -v /home/docker/redis/conf/redis.conf:/etc/redis/redis.conf -d redis redis-server /etc/redis/redis.conf
# 使用maclan 的方式启动
docker run --name redis --n -p 6379:6379 -v /home/docker/redis/data:/data -v /home/docker/redis/conf/redis.conf:/etc/redis/redis.conf -d redis redis-server /etc/redis/redis.conf
```

## 5 安装Nacos

```shell
docker pull chenfengwei/nacos:v1
docker run --env MODE=standalone --name nacos -d -p 8848:8848 chenfengwei/nacos:v1
docker cp nacos:/home/nacos /home/
```

## 6 安装Mysql

### 6.1 安装

1. 拉取镜像

   ```shell
   docker pull mysql/mysql-server
   ```

2. 运行mysql

   ```shell
   docker run -d -p 3306:3306 --name mysql mysql/mysql-server
   ```

3. 查看密码

   ```shell
   docker logs mysql
   ```

4. 进入容器

   ```shell
   docker exec -it mysql bash
   ```

5. 进入mysql的命令行

   ```shell
   mysql -uroot -p
   ```

6. 修改root密码

   ```shell
   use mysql;
   ALTER USER 'root'@'localhost' IDENTIFIED BY 'pwd123456';
   ```

### 6.2 创建用户与授权

```mysql
CREATE USER 'test'@'%' IDENTIFIED BY '123456';
GRANT ALL PRIVILEGES ON *.* TO 'test'@'%' WITH GRANT OPTION;

# 更改加密方式
ALTER USER 'root'@'%' IDENTIFIED BY '123456' PASSWORD EXPIRE NEVER;
# 重新修改密码
# navicat连接mysql报错1251解决方案
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
# 刷新
FLUSH PRIVILEGES;

```

#### 6.2.1 创建用户命令解释

`create user '<userName>'@'<host>' identified by '<passWord>';`

* userName 代表你要创建的此数据库的新用户账号

* host 代表访问权限，如下
  * %代表通配所有host地址权限(可远程访问)
  * localhost为本地权限(不可远程访问)

  * 指定特殊Ip访问权限 如10.138.106.102

* passWord 代表你要创建的此数据库的新用密码

```ruby
# 要创建的用户是testUser，密码是Haier...123,并且可远程访问
# 密码强度需要大小写及数字字母，否则会报密码强度不符合
# 用户名如果重复，会报错ERROR 1396 (HY000): Operation CREATE USER failed for 'testUser'@'%'
create user 'testUser'@'%' identified by 'Haier...123';
```

##### 6.2.1.1 查看用户

```shell
# 进入mysql系统数据库
user mysql;
# 查看用户的相关信息
select host, user, authentication_string, plugin from user;
```



#### 6.2.2 授权解释

`grant <auth> on <databaseName>.<table> to '<userName>'@'<host>;'`

* auth 代表权限，如下
  * all privileges 全部权限
  * select 查询权限
  * select,insert,update,delete 增删改查权限
  * select,[...]增...等权限
* databaseName 代表数据库名
* table 代表具体表，如下
  * `* ` 代表全部表
  * A,B 代表具体A,B表
* userName 代表用户名
* host 代表访问权限，如下
  * %代表通配所有host地址权限(可远程访问)
  * localhost为本地权限(不可远程访问)
  * 指定特殊Ip访问权限 如10.138.106.102

```ruby
# 赋予test用户对test数据库area_code表增删改差权限
grant select,insert,update,delete on test.area_code to 'test'@'%';
```

##### 6.2.2.1 查看权限

```shell
show grants for '#userName'@'#host';
#userName 代表用户名
#host 代表访问权限，如下
%代表通配所有host地址权限(可远程访问)
localhost为本地权限(不可远程访问)
指定特殊Ip访问权限 如10.138.106.102

# 查看的是testUser
show grants for 'testUser'@'%';
```

##### 6.2.2.2 撤销权限

`revoke <auth> on <databaseName>.<table> from '<userName>'@'<host>';`

* auth 代表权限，如下
  * all privileges 全部权限

  * select 查询权限

  * select,insert,update,delete 增删改查权限
  * select,[...]增...等权限

* databaseName 代表数据库名
  * table 代表具体表，如下
  * `*`代表全部表
  * A,B 代表具体A,B表

* userName 代表用户名

* host 代表访问权限，如下
  * %代表通配所有host地址权限(可远程访问)
  * localhost为本地权限(不可远程访问)
  * 指定特殊Ip访问权限 如10.138.106.102

```shell
# 要撤销testUser用户对b2b数据库中的area_code表的增删改差权限
revoke select,insert,update,delete on b2b.area_code from 'testUser'@'%';
```

```
# 查看用户权限
show grants for 'testUser'@'%';
```

### 6.3 修改MYSQL 默认字符集

1. 查看字符集命令

   ```ruby
   docker exec -it mysql01 bash
   mysql -uroot -ppwd123456
   use mysql;
   show variables like '%char%';
   +--------------------------+--------------------------------+
   | Variable_name            | Value                          |
   +--------------------------+--------------------------------+
   | character_set_client     | latin1                         |
   | character_set_connection | latin1                         |
   | character_set_database   | utf8mb4                        |
   | character_set_filesystem | binary                         |
   | character_set_results    | latin1                         |
   | character_set_server     | utf8mb4                        |
   | character_set_system     | utf8                           |
   | character_sets_dir       | /usr/share/mysql-8.0/charsets/ |
   +--------------------------+--------------------------------+
   ```

2. 修改my.cnf

   ```ruby
   [mysqld]
   character-set-server=utf8mb4
   [client]
   default-character-set=utf8mb4 
   [mysql]
   default-character-set=utf8mb4
   ```

   

### 6.4 docker容器参数启动Mysql

1. 创建容器

```shell
docker run -d -p 3306:3306 -e MYSQL_USER="admin" -e MYSQL_PASSWORD="123456" -e MYSQL_ROOT_PASSWORD="123456" -e MYSQL_ROOT_HOST=% --restart=always --name mysql mysql/mysql-server --character-set-server=utf8 --collation-server=utf8_general_ci
```

```shell
docker exec -it mysql bash 
mysql -uroot -ppwd123456
use mysql;
# navicat连接mysql报错1251解决方案
ALTER USER 'admin'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
# 刷新
FLUSH PRIVILEGES;
```

### 6.5 MySql挂载资料卷

```shell
#注意:需要先创建/home/docker/mysql/config/my.cnf文件和/home/docker/mysql/data文件夹
mkdir -p /home/docker/mysql/config
mkdir -p /home/docker/mysql/data
# 如果当前容器正在运行
docker cp mysql:/etc/my.cnf /home/docker/mysql/config/

# 或使用vim创建
vim config/my.cnf
[mysqld]
user=mysql
character-set-server = utf8	
# MySQL8默认的认证插件是caching_sha2_password，很多客户端都不支持，可将默认的认证插件修改为mysql_native_password，在配置文件中配置default_authentication_plugin=mysql_native_password。
default_authentication_plugin=mysql_native_password
character-set-server=utf8
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8

# mysql挂载资料卷
docker run -d -p 3306:3306 -v /home/docker/mysql/config/my.cnf:/etc/my.cnf -v /home/docker/mysql/data:/var/lib/mysql -e MYSQL_USER="admin" -e MYSQL_PASSWORD="123456" -e MYSQL_ROOT_PASSWORD="123456" -e MYSQL_ROOT_HOST=% --restart=always --name mysql mysql/mysql-server --character-set-server=utf8 --collation-server=utf8_general_ci
```

## 7 安装ES

镜像文档：https://hub.docker.com/r/arm64v8/elasticsearch

```shell
# 拉取镜像：
docker pull arm64v8/elasticsearch:7.11.1 (注意版本号根据文档上的，没有latest默认版本)
# 创建网络：
docker network create somenetwork 便于kibana等接入
# run镜像：
docker run -d --name es --net somenetwork -p 9200:9200 -p 9300:9300 -e ES_JAVA_OPTS="-Xms512m -Xmx512m" -e "discovery.type=single-node" arm64v8/elasticsearch:7.11.1 (Xms和Xmx根据实际设置，一般设置主存的一半)
```

## 8 安装MQ

```shell
docker pull rabbitmq
docker run -d --hostname my-rabbit --name rabbit -p 15672:15672 -p 5672:5672 rabbitmq

# 安装插件
# 先执行docker ps 拿到当前的镜像ID
# 进入容器
# 安装插件
# ctrl+p+q退出当前容器
docker exec -it 镜像ID /bin/bash
rabbitmq-plugins enable rabbitmq_management
用户名和密码默认都是guest
```

### 1 Stats in management UI are disabled on this node

```shell
#进入rabbitmq容器
docker exec -it {rabbitmq容器名称或者id} /bin/bash
#进入容器后，cd到以下路径
cd /etc/rabbitmq/conf.d/
#修改 management_agent.disable_metrics_collector = false
echo management_agent.disable_metrics_collector = false > management_agent.disable_metrics_collector.conf
#退出容器
exit
#重启rabbitmq容器
docker retart {rabbitmq容器id}
```

### 2 Web-UI界面无法访问 (同样遇到)

```shell
docker exec -it {rabbitmq容器名称或者id} rabbitmq-plugins enable rabbitmq_management
#重启rabbitmq容器
docker retart {rabbitmq容器id}
```

