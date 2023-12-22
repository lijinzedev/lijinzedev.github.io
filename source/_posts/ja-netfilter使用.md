---
title: ja-netfilter使用
top: false
cover: false
toc: true
mathjax: true
categories:
  - idea
tags:
  - idea
date: 2021-01-16 21:01:35
password:
summary:
---

## 下载ja-netfilter

你可以直接下载[ja-netfilter 2022.2](https://gitee.com/ja-netfilter/ja-netfilter/attach_files/1081163/download/ja-netfilter-2022.2.0.zip)或到[项目release页](https://gitee.com/ja-netfilter/ja-netfilter/releases)中寻找合适的版本

## 1 手动配置

### 配置JB产品的启动VM参数

使用合适的编辑器打开以下文件

```moonscript
C:\Users\admin\AppData\Roaming\JetBrains\产品名\产品名.exe.vmoptions
```

在文件尾部追加以下内容

```stylus
--add-opens=java.base/jdk.internal.org.objectweb.asm=ALL-UNNAMED
--add-opens=java.base/jdk.internal.org.objectweb.asm.tree=ALL-UNNAMED
//以上两行仅在Java17时需要添加，即JB产品的2022.2版本开始需要添加 同时本行注释需要删除
-javaagent:D:\soft\jetbra\ja-netfilter.jar=jetbrains
```

## 2 自动配置

<!--强调：该种方式不需要配置pycharm安装路径下的 bin\xxx.exe.vmoptions-->

**使用ja-netfilter-all\scripts下的配置脚本**

![image-20231222134056402](image-20231222134056402.png)

 	Windows系统就选`.vbs`后缀的，其他系统选择`.sh`后缀的。
`install-xxx`是安装，`uninstall-xxx`是卸载。

​	我这里是Windows系统，我要为系统中所有用户安装
​	双击`install-all-users.vbs`，弹框点击ok，安装完成即可。

**这个脚本做了哪些事情呢？**

1. 在ja-netfilter-all\vmoptions目录下的所有.vmoptions文件最后一行添加了--

   ```stylus
   javaagent:D:\Chen\MySoft\ja-netfilter-all\ja-netfilter.jar=jetbrains
   ```

   

2.  在环境变量中，为所有的JetBrains产品配置了启动VM有关的环境变量。

   ```stylus
   aJBProducts = Array("idea", "clion", "phpstorm", "goland", "pycharm", "webstorm", "webide", "rider", "datagrip", "rubymine", "appcode", "dataspell", "gateway", "jetbrains_client", "jetbrainsclient", "studio", "devecostudio")
   
   ```

![image-20231222134407360](image-20231222134407360.png)

当然如果你运行的是当前用户脚本，那么你的环境变量会在`用户变量`中。

## 3. 验证配置是否成功

在idea安装路径下的 bin目录，找到`idea.bat`，双击运行。
（如果你是其他产品，那它将是 `<产品名>.bat`，如 `idea.bat`）
出现如图所示，提示信息，证明配置成功了。

![image-20231222134507379](image-20231222134507379.png)

## 4. 注册产品

这里有两种方式：注册码，许可证服务器

### 方式一：使用注册码

访问：https://jetbra.in/s

你可能看到`August 1，2025`，会有疑惑有效期仅到 2025年？
再看下面一句话：`You have a perpetual fallback license for this version.`
就是说：您拥有此版本的永久备用许可。所以即使到期也不影响你继续使用。

### 方式二：使用许可证服务器

```stylus
https://jetbra.in
```

![image-20231222134644705](image-20231222134644705.png)

注意：有可能会失败，如果失败，你可能需要网上查找资料，重新配置config-jetbrains下的url.conf、dns.conf、power.conf 等配置，当然你也可以选择注册码方式。

## 注意

**本项目只做个人学习研究之用，不得用于商业用途！**

****
