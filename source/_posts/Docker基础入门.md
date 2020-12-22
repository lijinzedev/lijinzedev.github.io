---
title: Docker基础入门
top: false
cover: false
toc: true
mathjax: true
categories:
  - docker
tags:
  - docker
date: 2020-12-08 22:24:10
password:
summary:
---

# 1 Docker基础入门

## 1 安装

### 1 安装docker
```bash
#1.卸载旧版本
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine

#2.需要的安装包
yum install -y yum-utils

#3.设置镜像的仓库
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
#默认是从国外的，不推荐
#推荐使用国内的
yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

#更新yum软件包索引
yum makecache fast

#4.安装docker相关的(docker-ce 社区版 而ee是企业版)
yum install docker-ce docker-ce-cli containerd.io

#5.启动Docker
systemctl start docker

#6.使用docker version查看是否按照成功
docker version

#7.开机自启
systemctl enable docker
```

### 2 配置阿里云镜像加速
```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://4bj04jx5.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 2 命令

### 1 帮助命令

* `docker version`
* `docker info`
* `docker --help`

### 2  镜像命令

* `docker images` 列出本地的镜像
  * `-a ` 列出本地所有的镜像（含中间映像层）
  * `-q` 只显示镜像ID
  * `--digests` 显示镜像的摘要信息
  * `--no-trunc` 显示完整的镜像信息
  
* `docker search <某个镜像的名字>`  
  
  * `--no--trunc` 显示完整的镜像描述
  * `-s` 列出收藏数不小于指定值的镜像
  * `automated` 只列出automated build类型的镜像
  
* `docker pull <某个XXX镜像的名字>`
  
* `docker rmi <某个XXX镜像名字ID>` 
  
* 删除镜像
  * 删除单个 `docker rmi -f <镜像ID>`
  * 删除多个 `docker rmi -f 镜像名1:TAG 镜像名2:TAG`
  * 删除全部 `docker rmi -f ${docker images -qa}`
  * docker ps -a -q | xargs docker rm
  
  
  
###  3 容器命令

* `docker run <IMAGE>`
  * `--name=<容器的新名字>`
  * `-d` 后台运行容器,并返回容器ID,启动守护式容器
  * `-i `交互式
  * `-t `为容器重新分配一个伪输入终端
  * `-P` 随机端口映射
  * `-p `指定端口映射
    * ip:hostPort:containerPort
    * ip::containerPort
    * hostPort:containerPort
    * containerPort
* `docker run -it centos /bin/bash `

>   \#使用镜像centos:latest以交互模式启动一个容器,在容器内执行/bin/bash命令。 
>

* `docker ps` 列出所有**正在运行**的容器
  * `-a` :列出当前所有正在运行的容器+历史上运行过的
  * `-l` :显示最近创建的容器。
  * `-n`：显示最近n个创建的容器。
  * `-q` :静默模式，只显示容器编号。
  * `--no-trunc` :不截断输出。

* `exit` 容器停止推出
* `ctrk + P +Q` 容器不停止退出
* `docker start` 启动容器
* `docker stop `容器ID或者容器名
* `docker kill` 容器ID或者容器名
* `docker run -d centos /bin/sh -c "while true;do echo hello zzyy;sleep 2;done"`

> \#使用镜像centos:latest以后台模式启动一个容器
>
> docker run -d centos
>
>  
>
> 问题：然后docker ps -a 进行查看, 会发现容器已经退出
>
> 很重要的要说明的一点: Docker容器后台运行,就必须有一个前台进程.
>
> 容器运行的命令如果不是那些一直挂起的命令（比如运行top，tail），就是会自动退出的。
>
>  
>
> 这个是docker的机制问题,比如你的web容器,我们以nginx为例，正常情况下,我们配置启动服务只需要启动响应的service即可。例如
>
> service nginx start
>
> 但是,这样做,nginx为后台进程模式运行,就导致docker前台没有运行的应用,
>
> 这样的容器后台启动后,会立即自杀因为他觉得他没事可做了.
>
> 所以，最佳的解决方案是,将你要运行的程序以前台进程的形式运行

* `docker logs -f -t --tail <容器Id> `查看日志
* `docker top <容器Id>` 查看容器内进程
* `docker inspect <容器ID>` 查看容器内部细节
* `docker exec -it <容器ID> bashShell`
* `docker attach <容器ID> `

> attach 直接进入容器启动命令终端,不会启新的进程
>
> exec 是在容器打开新的终端,并且可以启动新的进程

* `docker cp 容器ID:路径 路径` 从容器拷贝到主机上



## 3 Docker 镜像

## 4 Docker数据卷

## 5 DockerFile

### 1 是什么？

> DockerFIle是用来构建Docker镜像的构建工具,是由一系列命令和参数构成的脚本

**构建三步:**

1. 编写DockerFile
2. docker build
3. docker run

### 2 DockerFIle构建解析

#### 1 DockerFIle的内容基础

1. 每条保留字指令都必须为大写字母且后面要跟随至少一个参数
2. 指令按照从上到下,顺序执行
3. \# 表示注释
4. **每条指令都会创建一个新的镜像曾,并对镜像进行提交**

#### 2 Docker执行Dockerfile的大致流程

1. docker从基础镜像运行一个容器
2. 执行一条指定并对容器做出修改
3. 执行类似docker commit 的操作提交一个新的镜像层
4. docker再基于刚提交的镜像运行一个新容器
5. 执行dockerfile的下一条指令知道所有指令都执行完成

### 3 Docker体系结构(保留字指令)

* `FORM` 基础镜像,当前镜像是基于那个镜像的
* `MAINTAINER` 镜像的作者与作者的邮箱
* `RUN` 容器构建需要执行的命令
* `EXPOSE`容器对外暴露的端口
* `WORKDIR`指定再创建容器后,终端默认登录进来的工作目录,一个落脚点
* `ENV`用于在构建镜像过程中设置环境变量
* `ADD` 将宿主机目录下的文件拷贝进镜像且ADD命令会自动处理URL和解压tar压缩包
* `COPY`类似于ADD 拷贝文件和目录到镜像中,将从构建上下文目录中<源路径>的文件/目录复制到新的一层的镜像内<目标路径>的位置
* `VOLUME`容器数据卷,用于数据保存和持久化工作
* `CMD` 指定一个容器启动时要运行的命令,DockerFIle中可以又多个CMD指令,但只有最后一个生效,CMD会被docker run的参数替换
* `ENTRYPOINT` 指定一个容器启动的时候要运行的命令,ENTRYPOINT的目的和CMD一样,都是在指定容器启动程序及参数

* `ONBUILD`当构建一个被继承的DockerFile时运行命令,父镜像在被子继承后,父镜像的onbuild被触发

| BUILD         | Both    | RUN        |
| ------------- | ------- | ---------- |
| FROM          | WORKDIR | CMD        |
| MAINTAINER    | USER    | ENV        |
| COPY          |         | EXPOSE     |
| ADD           |         | VOLUME     |
| RUN           |         | ENTRYPOINT |
| ONBUILD       |         |            |
| .dockerignore |         |            |

## 6 案例

### 1 BASE镜像(scratch)

> DockerHub中 99%的镜像都是通过在base镜像中安装和配置需要的软件构建出来的

### 2 自定义镜像mycentos

**需求:**

1. 登录后的默认路径

2. vim编辑器

3. 查看网络配置ifconfig支持

```dockerfile
FROM centos
ENV mypath /tmp
WORKDIR $mypath
RUN yum -y install vim
RUN yum -y install net-tools
EXPOSE 80
CMD /bin/bash
```

**构建:**

```bash
# . 表示上下文环境
docker build -t 新镜像名字:TAG .
# 运行
dokcer run -it 新镜像名字:TAG
# 列出镜像的变更历史
docker history 镜像名
```

### 3 ENTRYPOINT命令

**CMD**

> Dockerfile中可以有多个CMD指令,但只有最后一个生效,CMD会被Docker run 之后的参数替换

**ENTRYPOINT**

> docker run 之后的参数会被当作参数传递给ENTRYPOINT.之后形成新的命令组合

```dockerfile
ENTRYPOINT ["java","-java","test.jar"]
```

### 4 自定义镜像Tomcat

```dockerfile
FROM         centos
MAINTAINER    zzyy<zzyybs@126.com>
#把宿主机当前上下文的c.txt拷贝到容器/usr/local/路径下
COPY c.txt /usr/local/cincontainer.txt
#把java与tomcat添加到容器中
ADD jdk-8u171-linux-x64.tar.gz /usr/local/
ADD apache-tomcat-9.0.8.tar.gz /usr/local/
#安装vim编辑器
RUN yum -y install vim
#设置工作访问时候的WORKDIR路径，登录落脚点
ENV MYPATH /usr/local
WORKDIR $MYPATH
#配置java与tomcat环境变量
ENV JAVA_HOME /usr/local/jdk1.8.0_171
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.8
ENV CATALINA_BASE /usr/local/apache-tomcat-9.0.8
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin
#容器运行时监听的端口
EXPOSE  8080
#启动时运行tomcat
# ENTRYPOINT ["/usr/local/apache-tomcat-9.0.8/bin/startup.sh" ]
# CMD ["/usr/local/apache-tomcat-9.0.8/bin/catalina.sh","run"]
CMD /usr/local/apache-tomcat-9.0.8/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.8/
```

**运行**

```bash
docker run -d -p 9080:8080 --name myt9 -v /zzyyuse/mydockerfile/tomcat9/test:/usr/local/apache-tomcat-9.0.8/webapps/test -v /zzyyuse/mydockerfile/tomcat9/tomcat9logs/:/usr/local/apache-tomcat-9.0.8/logs --privileged=true zzyytomcat9

```

## 7 Docker网络

![image-20201222233356901](image-20201222233356901.png)

* 容器的网卡是一对一对的
* evth-pair，就是一对虚拟设备接口，他们都是成对出现的，一段连着协议，一段彼此相连
* evth-pair充当一个桥梁，连接各种虚拟网络

> 所有的容器不指定网络的情况下,都是docker0,docker会给容器分配一个默认的IP

## 8 容器互联--link（不推荐）



```bash
# --link 连接网络,centos 可以ping通tomcat 反之不行，在host的配置中添加了tomcat的映射
docker run -d -P --name centos --link tomcat
```

## 9 自定义网络

* brideg 桥接（默认）自定义方式
* none 不配置网络
* host 和宿主机共享网络
* container 容器网络连通（局限性大）

```bash
[root@localhost ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
b1641c6fe2b8        bridge              bridge              local
6e94609c9ec5        host                host                local
13cba1d43999        none                null                local
[root@localhost ~]# 

```

```bash
# 直接启动的命令默认有一个 --net bridge  这个就是docker0
docker run -d -P --name centos --net bridge centos
# docker0 特点，默认，域名不能访问，--link可以打通连接

docker network create --driver bridge --subnet 192.168.0.1/16 --gateway 192.168.0.1 customnet

# 查看网络
docker network inspect customnet

```

> 自定义网络docker可以帮助维护对应关系，推荐平时使用
>
> 不同的集群可以使用不同的网络，保证集群是安全和健康的

### 自定义网络连通docker0或者其他网络

```bash
docker network connect customnet centos
#连通之后就是把 centos 与customnet 打通，将centos放到了customnet网络下，也就是一个容器两个ip
```

跨网络操作其他容器就需要使用docker network connect

## 10 Redis集群

```java
docker network create --driver bridge --subnet 192.168.0.1/16 --gateway 192.168.0.1 redis
```

S