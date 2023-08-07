---
title: Linux 环境下查看日志
top: false
cover: false
toc: true
mathjax: true
categories:
  - linux
  - 日志
tags:
  - linux
date: 2022-12-11 09:43:07
password:
summary:
---

# 一、查看日志

- `cat service.log`
- `tail -f service.log`
- `vim serivice.log`
  - 按`G`跳转到文件的末尾
  - 按`?` +关键字搜索对应的记录
  - 按`n`往上查询，按`N`往下查询

> 面对大文件时候使用grep

比如我们现在得知某个手机号收不到短信验证码，想要看一下这个手机号的日志是怎么样的。于是我们就可以这样搞：

- `cat -n service.log | grep 13888888888`

这样，就能将`service.log`中所有含有`13888888888`的记录给搜出来。

**日志上下文**

- `sed -n "29496,29516p" service.log`：从29496行开始检索，到29516行结束
- `cat -n service.log | tail -n +29496 | head -n 20`:从29496行开始检索，往前推20条

如果关键字不太准确（日志输出的记录太多了），我们可以使用`more`命令来浏览或者输出到文件上再分析：

- `cat service.log | grep 13 |more` ：将查询后的结果交由more输出
- `cat service.log | grep 13 > /home/aa.txt` 将查询后的结果写到`/home/aa.txt`文件上

有的时候，我们想统计这个日志输出了多少行，我们可以使用这条命令：

- `cat service.log | wc -l`
- `cat filename |grep 关键字 -C10 `上面显示关键字的前后10行          -C显示前后多少行
- `cat filename |grep 关键字 -A10`  上面显示关键字的后10行              -A显示后多少行
- `cat filename |grep 关键字 -B10  `上面显示关键字的前10行              -B显示前多少行



## 参考文档

[](https://www.cnblogs.com/xiashan17/p/7059978.html)

# 二、docker-compose 命令翻译

```bash
 # 登录到nginx容器中
docker-compose exec nginx bash            
# 删除所有nginx容器,镜像
docker-compose down    
# 重新启动nginx容器
docker-compose restart nginx                   
# 在php-fpm中不启动关联容器，并容器执行php -v 执行完成后删除容器
docker-compose run --no-deps --rm php-fpm php -v 
# 构建镜像 
docker-compose build nginx                      
# 不带缓存的构建。
docker-compose build --no-cache nginx  
# 查看nginx的日志 
docker-compose logs  nginx                  
# 查看nginx的实时日志
docker-compose logs -f nginx                  
#  验证（docker-compose.yml）文件配置，当配置正确时，不输出任何内容，当文件配置错误，输出错误信息
docker-compose config  -q                       。 
# 以json的形式输出nginx的docker日志
docker-compose events --json nginx       
# 暂停nignx容器
docker-compose pause nginx             
# 恢复ningx容器
docker-compose unpause nginx                     
# 停止nignx容器
docker-compose stop nginx                
# 启动nignx容器
docker-compose start nginx               
```

## rm

```bash
rm
 
`docker-compose rm` 删除服务（停止状态）容器。
 
# 删除所有（停止状态）服务的容器
 
docker-compose rm
 
# 先停止所有服务的容器，再删除所有服务的容器
 
docker-compose rm -s
 
# 不询问是否删除，直接删除
 
docker-compose rm -f
 
# 删除服务容器挂载的数据卷
 
docker-compose rm -v
 
# 删除工程中指定服务的容器
 
docker-compose rm -sv nginx
```

## up

```bash
#在后台运行服务容器
docker-compose -d 
# 不使用颜色来区分不同的服务的控制输出
docker-compose –no-color 
# 不启动服务所链接的容器
docker-compose –no-deps 
#  强制重新创建容器，不能与–no-recreate同时使用
docker-compose –force-recreate
# 如果容器已经存在，则不重新创建，不能与–force-recreate同时使用
docker-compose –no-recreate 
# 不自动构建缺失的服务镜像
docker-compose  –no-build 
# 在启动容器前构建服务镜像
docker-compose –build 
# 停止所有容器，如果任何一个容器被停止，不能与-d同时使用
docker-compose –abort-on-container-exit 
# 停止容器时候的超时（默认为10秒）
docker-compose -t, –timeout TIMEOUT
# 删除服务中没有在compose文件中定义的容器
docker-compose –remove-orphans 
# 设置服务运行容器的个数，将覆盖在compose中通过scale指定的参数
docker-compose –scale SERVICE=NUM 
# 指定使用的Compose模板文件，默认为docker-compose.yml，可以多次指定。
docker-compose -f 
```



```bash
# 列出当docker-compose 服务service名称
docker-compose ps --services
# 列出当docker-compose 容器名称
docker-compose ps 
# 列出当docker-compose 容器id
docker-compose ps  -q

docker-compose logs -f    --tail="all" -- backend-dynamic-mining-backend 
```

