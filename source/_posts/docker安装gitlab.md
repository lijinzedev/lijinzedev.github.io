---
title: docker安装gitlab
top: false
cover: false
toc: true
mathjax: true
categories:
  - linux - CentOS
tags:
  - linux - CentOS - gitlab - docker
date: 2020-11-29 16:10:36
password:
summary:
---

[TOC]

# Docker安装Gitlab
参考：

[参阅链接](https://blog.csdn.net/qq_33619378/article/details/89706222)

[参阅链接](https://www.cnblogs.com/zuxing/articles/9329152.html)

[参阅链接](https://segmentfault.com/a/1190000021229534)

## 安装docker
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

## 配置阿里云镜像加速
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
## 准备挂载目录bash
```bash
通常会将 GitLab 的配置 (etc) 、 日志 (log) 、数据 (data) 放到容器之外， 便于日后升级， 因此请先准备这三个目录。


mkdir -p /home/gitlab/etc
mkdir -p /home/gitlab/log
mkdir -p /home/gitlab/data
```
## 安装gitlab
方式一、在线下载gitlab
```bash
拉取镜像
docker pull gitlab/gitlab-ce:latest

```
方式二、离线安装
```bash
在事先安装好的服务器上，打包镜像
docker save -o E:\gitlab-ce-latest.tar gitlab/gitlab-ce:latest

然后拷贝到目标服务器上，docker加载镜像：tar->image

1.从tar包载入镜像：

docker load -i {image_name}.tar

2.查看载入是否成功：

docker images | grep {image_name}

3.如果看到加载的镜像没有tag和镜像名，则手动打tag:

docker tag {image_id} {image_name}:{image_tag}

4.确认镜像是否成功打上tag
```
## 启动容器，指定端口及挂载
```bash
docker run -d \
-p 8002:443 \
-p 8001:80 \
-p 8000:22 \
--name gitlab \
--restart always \
--hostname 192.168.0.100 \
-v /home/gitlab/etc:/etc/gitlab  \
-v /home/gitlab/logs:/var/log/gitlab  \
-v /home/gitlab/data:/var/opt/gitlab  \
gitlab/gitlab-ce

```

## 修改gitlab配置
```bash
1、修改/home/gitlab/etc/gitlab.rb

把external_url改成部署机器的域名或者IP地址

vi /home/gitlab/etc/gitlab.rb

# 配置http协议所使用的访问地址
external_url 'http://172.16.7.110:8001'
# 配置ssh协议所使用的访和端口
gitlab_rails['gitlab_shell_ssh_port'] = 8002

# nginx 的监听端口号需要改成 80。
# 默认情况下 nginx 的监听端口号会从 external_url 中取，也就是 9080。
# 但在启动容器时，我们把宿主机 9080 端口导向了容器的 80 端口，所以容器内 nginx 服务端口应该为 80。
nginx['listen_port'] = 80

gitlab_rails['time_zone'] = 'Asia/Shanghai'
2、修改/home/gitlab/data/gitlab-rails/etc/gitlab.yml
vi /home/gitlab/data/gitlab-rails/etc/gitlab.yml

找到关键字 * ## Web server settings * 

将host的值改成映射的外部主机ip地址和端口，这里会显示在gitlab克隆地址

```
## 重启容器
```bash
改完后，重启docker容器。先停止该容器，删掉该容器信息，重启完docke之后，重新运行GitLab容器

查询容器列表
docker ps

停止容器
docker stop 容器id

删除容器
doccker rm 容器id

查询容器列表
docker ps

重启
systemctl restart docker

启动gitlab
docker run -d     --hostname 172.16.7.14     --publish 7001:443 --publish 7002:80 --publish 7003:22     --name gitlab --restart always     --volume /home/gitlab/etc:/etc/gitlab     --volume /home/gitlab/logs:/var/log/gitlab     --volume /home/gitlab/data:/var/opt/gitlab 镜像id


关闭防火墙（选做）
systemctl stop firewalld.service

如果不关闭防火墙，还可以通过暴露指定端口
firewall-cmd --permanent --add-port=7002/tcp
firewall-cmd --reload
```
## 开始访问
Gitlab容器启动后，直接访问 http://ip  就可以进入gitlab访问页面，第一步要做的就是给root用户设置密码，设置完后，通过root + 设置的密码登录。


## 其他
> 常用的几个Gitlab命令
```bash
# 重新应用gitlab的配置
gitlab-ctl reconfigure
 
# 重启gitlab服务
gitlab-ctl restart
 
# 查看gitlab运行状态
gitlab-ctl status
 
#停止gitlab服务
gitlab-ctl stop
 
# 查看gitlab运行日志
gitlab-ctl tail
 
# 停止相关数据连接服务
gitlab-ctl stop unicorn
gitlab-ctl stop sideki
```

> 修改了GitLab的ip地址一样，临时修改了GitLab的配置之后，得执行如下的命令，应用重新配好的配置并重启GitLab，然后查看GitLab的状态。

因为是容器，所以要进入到gitlab容器中执行命令
```bash
进入容器
docker exec -ti gitlab /bin/bash

容器内部执行命令
gitlab-ctl reconfigure  #花时间比较多
gitlab-ctl restart    #改IP重启就可以了
gitlab-ctl status
```

