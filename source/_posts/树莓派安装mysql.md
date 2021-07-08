---
title: 树莓派安装mysql
top: false
cover: false
toc: true
mathjax: true
categories:
  - 树莓派
tags:
  - 树莓派
date: 2021-07-04 00:11:18
password:
summary:
---

# 树莓派Docker 安装mysql

```bash
# 拉镜像
docker pull hypriot/rpi-mysql:latest
# 复制配置
docker cp  mysql:/etc/mysql /home/docker/mysql
# 删除镜像
docker rm -f mysql
# 启动镜像并且映射
sudo docker run -p 3307:3306 --name mysql \
    -v /home/mysql/mysql:/etc/mysql \
    -v /home/mysql/log:/var/log/mysql \
    -v /home/mysql/data:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD=123456 \
    -d hypriot/rpi-mysql
# 进入容器
docker exec -it mysql /bin/bash
# 输入
mysql -u root -p
# 创建远程登录用户和权限设置
use mysql;
CREATE USER 'admin'@'%' IDENTIFIED BY 'admin';
ALTER USER 'admin'@'%' IDENTIFIED WITH mysql_native_password BY 'admin';
GRANT ALL ON *.* TO 'admin'@'%';
flush privileges;
# 刷新root
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

