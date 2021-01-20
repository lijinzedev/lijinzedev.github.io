---
title: GitHub Action 入门
top: false
cover: false
toc: true
mathjax: true
categories:
  - CI/CD
tags:
  - CI/CD
date: 2021-01-06 16:35:02
password:
summary:
---



# 1 入门

## 1 官方示例

```yaml

# 流程命名
name: CI 
# 决定什么时候触发
on:
# push的时候触发
  push:
    branches: [ master ]
# 具体的jobs
jobs:
# 命名为build 也可以是其他的
  build:
  # 在ubuntu-latest系统下运行，github提供了默认的几个环境运行
    runs-on: ubuntu-latest
    # 执行具体的步骤，每一步是一个数组
    steps:
    # 拉代码
      - uses: actions/checkout@v2
      - name: Run a one-line script
        # 运行单行脚本
        run: echo Hello, world!
      - name: Run a multi-line script
        # 运行多行脚本
        run: |
          echo Add other actions to build,
          echo test, and deploy your project.

```

## 2 各个关键字简单解释

* `uses`

> run-on指定的运行环境中，已经默认安装了很多工具的版本，比如node.js 如果不指定版本，会默认使用指定运行环境中的版本
>
> ```yaml
> - uses: actions/setup-node@v1
>   with:
>   # 指定版本
>     node-version: 8
> ```

```yaml
# action的名字
name: My-First-Ci
# on : [push]或
# src目录下任何文件发生改动的主分支提交代码触发流程
on:
  push:
    branches:
    - master
    paths:
    - src/*
jobs:
  job1:
  # runs-on 是枚举类型，八种之一
    runs-on: ubuntu-latest
    # 配置策略
    strategy:
      matrix:
      # 配置策略分别执行测试
        node-version: [12.x]
    steps:
    # 使用uses下载代码
      uses: actions/checkout@v2
      uses: actions/setup-node@v1
		with:
		  # 指定版本
          node-version: 8
    # 下载代码
    - run: git clone xxx 
    # run 运行shell指令
    - run: echo hello
  job2:
  # job2 执行需要job1执行完成
    needs:job1
	
		
  
```





# 2 实战

## 1 GitHub Action 部署前端项目到云服务Docker上

###  1 准备DOCKERFILE

```dockerfile
FROM nginx
COPY ./dist/ /usr/share/nginx/html/
COPY ./nginx/default.conf /etc/nginx/conf.d/default.conf
```

### 2  准备nginx.conf

```nginx
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    access_log  /var/log/nginx/host.access.log  main;
    error_log  /var/log/nginx/error.log  error;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

### 3 创建镜像仓库 

> 在阿里云中创建Docker镜像仓库，并记录公网ip地址

### 4 创建Secrets

> 登录github进入项目仓库，依次点击settings>Secrets>New secret

> 点击New secret以后出来的页面有2个选项，Name和Value，Name对应上图红框所示，Value填入Name对应的值，简单解释一下：

```
DOCKER_REPOSITORY: 镜像仓库地址，也就是上一个步骤复制到的公网地址

DOCKER_USERNAME：登录阿里云的账号

DOCKER_PASSWORD： 登录阿里云的密码

HOST：部署项目的服务器ip

HOST_PORT：服务器ssh端口号(默认是22)

HOST_USERNAME：服务器登录用户名（ps:非root权限账号请子u该账号所属组为docker）

HOST_PASSWORD： 登录服务器的密码
```



### 5  准备Workflows.yml文件



```yaml
name: Vue-Test-CI
on:
  push:
    branches:
      - master
jobs:
  build:
    runs-on: ubuntu-latest
	# 多策略的用法
    strategy:
      matrix:
        node-version: [12.x]

    steps:
    - uses: actions/checkout@v2
    - name: Use Node.js 8
    - uses: actions/setup-node@v1
      with:
        node-version: 8
    - name: npm install, build, and test
      run: |
        npm install
        npm run build:prod
    - name: Build Image
      run: docker build -t ${{ secrets.DOCKER_REPOSITORY }}:latest ./
    - name: Login to registry
      run: docker login --username=${{ secrets.DOCKER_USERNAME }} --password ${{ secrets.DOCKER_PASSWORD }} registry.cn-hangzhou.aliyuncs.com
    - name: Push Image
      run: docker push ${{ secrets.DOCKER_REPOSITORY }}:latest
  pull-docker:
    needs: [build]
    name: Pull Docker
    runs-on: ubuntu-latest
    steps:
      - name: Deploy
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.HOST_USERNAME }}
          password: ${{ secrets.HOST_PASSWORD }}
          port: ${{ secrets.HOST_PORT }}
          script: |
            docker stop $(docker ps --filter ancestor=${{ secrets.DOCKER_REPOSITORY }} -q)
            docker rm -f $(docker ps -a --filter ancestor=${{ secrets.DOCKER_REPOSITORY }}:latest -q)
            docker rmi -f $(docker images ${{ secrets.DOCKER_REPOSITORY }}:latest -q)
            docker login --username=${{ secrets.DOCKER_USERNAME }} --password ${{ secrets.DOCKER_PASSWORD }} registry.cn-hangzhou.aliyuncs.com
            docker pull ${{ secrets.DOCKER_REPOSITORY }}:latest
            docker run -d -p 8000:80 ${{ secrets.DOCKER_REPOSITORY }}:latest
```

### 6  注释版

```yaml
name: Docker Image CI/CD # workflow名称，可以随意改
on: # workflow的事件钩子，告知程序说明时候出发自动部署
  push:
    branches: [ master ] # 在master分支有push操作的时候自动部署
jobs:
  build: # 打包并上传docker镜像
    runs-on: ubuntu-latest # 依赖的环境      
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v1
      with:
        node-version: 8
      - name: npm install, build, and test
      run: |
      # 安装依赖
        npm install
        # 构建
        npm run build:prod 
      - name: Build Image
      # ${{ secrets.DOCKER_REPOSITORY }}是读取之前在Secret创建的名为DOCKER_REPOSITORY的值
        run: docker build -t ${{ secrets.DOCKER_REPOSITORY }}:latest ./ # 打包并docker镜像，版本为latest
      - name: Login to Registry # 登录阿里云镜像服务器
        run: docker login --username=${{ secrets.DOCKER_USERNAME }} --password ${{ secrets.DOCKER_PASSWORD }} registry.cn-hangzhou.aliyuncs.com
      - name: Push Image # 推送镜像，设置版本为latest
        run: docker push ${{ secrets.DOCKER_REPOSITORY }}:latest
  pull-docker: # docker部署
    needs: [build]
    name: Pull Docker
    runs-on: ubuntu-latest
    steps:
      - name: Deploy
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.HOST }} # 服务器ip
          username: ${{ secrets.HOST_USERNAME }} # 服务器登录用户名
          password: ${{ secrets.HOST_PASSWORD }} # 服务器登录密码
          port: ${{ secrets.HOST_PORT }} # 服务器ssh端口
          script: |
          	# 停止旧版容器
            docker stop $(docker ps --filter ancestor=${{ secrets.DOCKER_REPOSITORY }} -q)
            # 删除旧版容器
            docker rm -f $(docker ps -a --filter ancestor=${{ secrets.DOCKER_REPOSITORY }}:latest -q)
            # 删除旧版镜像
            docker rmi -f $(docker images ${{ secrets.DOCKER_REPOSITORY }}:latest -q)
            # 登录阿里云镜像服务器
            docker login --username=${{ secrets.DOCKER_USERNAME }} --password ${{ secrets.DOCKER_PASSWORD }} registry.cn-hangzhou.aliyuncs.com
            # 拉取最新latest版本镜像
            docker pull ${{ secrets.DOCKER_REPOSITORY }}:latest
            # 运行最新latest版本镜像
            docker run -d -p 8000:4000 ${{ secrets.DOCKER_REPOSITORY }}:latest


```



## 2 Github Action 部署maven项目到云服务Docker上

3 Github Action 部署前后端分离Cloud项目云服务Docker上

```yaml
name: auto-deploy-gatway
on:
  push:
    branches:
      - master
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: 拉取代码
      uses: actions/checkout@v2
    - name: 准备jdk8 环境
      uses: actions/setup-java@v1
      with:
        java-version: 1.8
    - name: 打包后端
      run: mvn -B -Dmaven.test.skip=true package --file pom.xml
    - name: 构建镜像
      run: docker build -f ./ruoyi-gateway/Dockerfile -t ${{ secrets.DOCKER_REPOSITORY }}/ruoyi-gateway:latest ./ruoyi-gateway
    - name: 登录到阿里云镜像
      run: docker login --username=${{ secrets.DOCKER_USERNAME }} --password ${{ secrets.DOCKER_PASSWORD }} registry.cn-hangzhou.aliyuncs.com
    - name: 推送镜像
      run: docker push ${{ secrets.DOCKER_REPOSITORY }}/ruoyi-gateway:latest
  pull-docker:
    needs: [build]
    name: Pull Docker
    runs-on: ubuntu-latest
    steps:
      - name: 部署后端
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.HOST_USERNAME }}
          password: ${{ secrets.HOST_PASSWORD }}
          port: ${{ secrets.HOST_PORT }}
          script: |
            docker stop $(docker ps --filter ancestor=${{ secrets.DOCKER_REPOSITORY }}/ruoyi-gateway -q)
            docker rm -f $(docker ps -a --filter ancestor=${{ secrets.DOCKER_REPOSITORY }}/ruoyi-gateway:latest -q)
            docker rmi -f $(docker images ${{ secrets.DOCKER_REPOSITORY }}/ruoyi-gateway:latest -q)
            docker login --username=${{ secrets.DOCKER_USERNAME }} --password ${{ secrets.DOCKER_PASSWORD }} registry.cn-hangzhou.aliyuncs.com
            docker pull ${{ secrets.DOCKER_REPOSITORY }}/ruoyi-gateway:latest
            docker run -d --name ruoyi-gateway -p 8080:8080 ${{ secrets.DOCKER_REPOSITORY }}/ruoyi-gateway:latest
```

## 3 Github Action 部署微服务项目到服务器Docker上

> 整合 1 与 2

```yaml
name: auto-deploy-gatway
on:
  push:
    branches:
      - master
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: 拉取代码
      uses: actions/checkout@v2
    - name: 准备jdk8 环境
      uses: actions/setup-java@v1
      with:
        java-version: 1.8
    - name: 打包后端
      run: mvn -B -Dmaven.test.skip=true package --file pom.xml
    - name: 构建网关镜像
      run: docker build -f ./ruoyi-gateway/Dockerfile -t ${{ secrets.DOCKER_REPOSITORY }}/ruoyi-gateway:latest ./ruoyi-gateway
    - name: 构建服务镜像
      run: docker build -f ./ruoyi-modules/ruoyi-system/Dockerfile -t ${{ secrets.DOCKER_REPOSITORY }}/ruoyi-system:latest ./ruoyi-modules/ruoyi-system
    - name: 登录到阿里云镜像
      run: docker login --username=${{ secrets.DOCKER_USERNAME }} --password ${{ secrets.DOCKER_PASSWORD }} registry.cn-hangzhou.aliyuncs.com
    - name: 推送网关镜像
      run: docker push ${{ secrets.DOCKER_REPOSITORY }}/ruoyi-gateway:latest
    - name: 推送微服务镜像
      run: docker push ${{ secrets.DOCKER_REPOSITORY }}/ruoyi-system:latest      
  pull-docker:
    needs: [build]
    name: Pull Docker
    runs-on: ubuntu-latest
    steps:
      - name: 部署后端
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.HOST_USERNAME }}
          password: ${{ secrets.HOST_PASSWORD }}
          port: ${{ secrets.HOST_PORT }}
          script: |
            docker stop $(docker ps --filter ancestor=${{ secrets.DOCKER_REPOSITORY }}/ruoyi-gateway -q)
            docker rm -f $(docker ps -a --filter ancestor=${{ secrets.DOCKER_REPOSITORY }}/ruoyi-gateway:latest -q)
            docker rmi -f $(docker images ${{ secrets.DOCKER_REPOSITORY }}/ruoyi-gateway:latest -q)
            docker login --username=${{ secrets.DOCKER_USERNAME }} --password ${{ secrets.DOCKER_PASSWORD }} registry.cn-hangzhou.aliyuncs.com
            docker pull ${{ secrets.DOCKER_REPOSITORY }}/ruoyi-gateway:latest
            docker run -d --name ruoyi-gateway -p 8080:8080 ${{ secrets.DOCKER_REPOSITORY }}/ruoyi-gateway:latest
            docker stop $(docker ps --filter ancestor=${{ secrets.DOCKER_REPOSITORY }}/ruoyi-system -q)
            docker rm -f $(docker ps -a --filter ancestor=${{ secrets.DOCKER_REPOSITORY }}/ruoyi-system:latest -q)
            docker rmi -f $(docker images ${{ secrets.DOCKER_REPOSITORY }}/ruoyi-system:latest -q)
            docker pull ${{ secrets.DOCKER_REPOSITORY }}/ruoyi-system:latest
            docker run -d --name ruoyi-system -p 9201:9201 ${{ secrets.DOCKER_REPOSITORY }}/ruoyi-system:latest
```



# 3 参阅

[Docker参阅](https://blog.csdn.net/alangrady/article/details/108241799)

[入门参阅](https://yuqingc.github.io/posts/2020/github-actions/)