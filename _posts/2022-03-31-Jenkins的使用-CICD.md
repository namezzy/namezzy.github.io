---
layout: post
title: Jenkins的使用-CI/CD
subtitle: CI/CD
date: 2022-03-31
author: Levi
header-img: img/CICD/CICD.png
catalog: true
top: false
tags:
  - CI/CD
  - 持续集成 持续部署
  - DevOps

---



## Jenkins的使用-CI/CD

### 一、去Jenkins官网下载war包

[下载](https://www.jenkins.io/download/)

> jenkins.war

![image-20220331203820293](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/image-20220331203820293.png)



### 二、打开终端运行命令

```bash
java -jar jenkins.war
```



运行结果:

```bash
➜ ~ java -jar /Users/levi.zhao/Desktop/jenkins.war
Running from: /Users/levi.zhao/Desktop/jenkins.war
webroot: $user.home/.jenkins
2022-03-31 12:44:55.771+0000 [id=1]	INFO	org.eclipse.jetty.util.log.Log#initialized: Logging initialized @622ms to org.eclipse.jetty.util.log.JavaUtilLog
2022-03-31 12:44:55.870+0000 [id=1]	INFO	winstone.Logger#logInternal: Beginning extraction from war file
2022-03-31 12:44:55.915+0000 [id=1]	WARNING	o.e.j.s.handler.ContextHandler#setContextPath: Empty contextPath
2022-03-31 12:44:55.994+0000 [id=1]	INFO	org.eclipse.jetty.server.Server#doStart: jetty-9.4.43.v20210629; built: 2021-06-30T11:07:22.254Z; git: 526006ecfa3af7f1a27ef3a288e2bef7ea9dd7e8; jvm 1.8.0_281-b09
2022-03-31 12:44:56.509+0000 [id=1]	INFO	o.e.j.w.StandardDescriptorProcessor#visitServlet: NO JSP Support for /, did not find org.eclipse.jetty.jsp.JettyJspServlet
2022-03-31 12:44:56.576+0000 [id=1]	INFO	o.e.j.s.s.DefaultSessionIdManager#doStart: DefaultSessionIdManager workerName=node0
2022-03-31 12:44:56.576+0000 [id=1]	INFO	o.e.j.s.s.DefaultSessionIdManager#doStart: No SessionScavenger set, using defaults
2022-03-31 12:44:56.577+0000 [id=1]	INFO	o.e.j.server.session.HouseKeeper#startScavenging: node0 Scavenging every 600000ms
2022-03-31 12:44:57.169+0000 [id=1]	INFO	hudson.WebAppMain#contextInitialized: Jenkins home directory: /Users/levi.zhao/.jenkins found at: $user.home/.jenkins
2022-03-31 12:44:58.733+0000 [id=1]	INFO	o.e.j.s.handler.ContextHandler#doStart: Started w.@28d18df5{Jenkins v2.332.1,/,file:///Users/levi.zhao/.jenkins/war/,AVAILABLE}{/Users/levi.zhao/.jenkins/war}
2022-03-31 12:44:58.793+0000 [id=1]	INFO	o.e.j.server.AbstractConnector#doStart: Started ServerConnector@6ea12c19{HTTP/1.1, (http/1.1)}{0.0.0.0:8080}
2022-03-31 12:44:58.793+0000 [id=1]	INFO	org.eclipse.jetty.server.Server#doStart: Started @3645ms
2022-03-31 12:44:58.794+0000 [id=23]	INFO	winstone.Logger#logInternal: Winstone Servlet Engine running: controlPort=disabled
2022-03-31 12:45:00.880+0000 [id=30]	INFO	jenkins.InitReactorRunner$1#onAttained: Started initialization
2022-03-31 12:45:00.883+0000 [id=35]	INFO	jenkins.InitReactorRunner$1#onAttained: Listed all plugins
2022-03-31 12:45:02.302+0000 [id=28]	INFO	jenkins.InitReactorRunner$1#onAttained: Prepared all plugins
2022-03-31 12:45:02.313+0000 [id=30]	INFO	jenkins.InitReactorRunner$1#onAttained: Started all plugins
2022-03-31 12:45:02.327+0000 [id=30]	INFO	jenkins.InitReactorRunner$1#onAttained: Augmented all extensions
2022-03-31 12:45:03.858+0000 [id=31]	INFO	jenkins.InitReactorRunner$1#onAttained: System config loaded
2022-03-31 12:45:03.859+0000 [id=35]	INFO	jenkins.InitReactorRunner$1#onAttained: System config adapted
2022-03-31 12:45:03.859+0000 [id=35]	INFO	jenkins.InitReactorRunner$1#onAttained: Loaded all jobs
2022-03-31 12:45:03.864+0000 [id=35]	INFO	jenkins.InitReactorRunner$1#onAttained: Configuration for all jobs updated
2022-03-31 12:45:03.943+0000 [id=49]	INFO	hudson.model.AsyncPeriodicWork#lambda$doRun$1: Started Download metadata
2022-03-31 12:45:03.956+0000 [id=49]	INFO	hudson.util.Retrier#start: Attempt #1 to do the action check updates server
2022-03-31 12:45:04.607+0000 [id=36]	INFO	jenkins.install.SetupWizard#init:

*************************************************************
*************************************************************
*************************************************************

Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:
//本地生成的密码
294767bf59404c0bae62ac808f136d2f

This may also be found at: /Users/levi.zhao/.jenkins/secrets/initialAdminPassword

*************************************************************
*************************************************************
*************************************************************

2022-03-31 12:45:25.198+0000 [id=28]	INFO	jenkins.InitReactorRunner$1#onAttained: Completed initialization
2022-03-31 12:45:25.228+0000 [id=22]	INFO	hudson.lifecycle.Lifecycle#onReady: Jenkins is fully up and running
2022-03-31 12:45:26.785+0000 [id=49]	INFO	h.m.DownloadService$Downloadable#load: Obtained the updated data file for hudson.tasks.Maven.MavenInstaller
2022-03-31 12:45:26.785+0000 [id=49]	INFO	hudson.util.Retrier#start: Performed the action check updates server successfully at the attempt #1
2022-03-31 12:45:26.793+0000 [id=49]	INFO	hudson.model.AsyncPeriodicWork#lambda$doRun$1: Finished Download metadata. 22,849 ms
```



### 三、打开浏览器输入http://localhost:8080/

1.  把终端生成的密码-294767bf59404c0bae62ac808f136d2f  输入进去

   ![image-20220331204904508](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/image-20220331204904508-20220331205034176.png)

   
   
2. 自定义Jenkins

   选择-安装推荐的插件

   ![image-20220331205151431](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/image-20220331205151431.png)

   

3. 开始自动下载所需的插件(需要等一段时间)

   ![image-20220331205336684](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/image-20220331205336684.png)

   

   

4. 创建第一个管理员用户

   ![image-20220331205810111](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/image-20220331205810111.png)

   

   

5. 进入到Jenkins主页面

   ![image-20220331205950099](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/image-20220331205950099.png)
   



 

### 四、创建项目

1. 选择 Freestyle project

   ![image-20220331210340486](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/image-20220331210340486.png)
   
2. 先使用github创建一个仓库

   ![](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/image-20220331210547037.png)
   
3. 然后把仓库地址填写到 Repository URL中

   ![image-20220331210846540](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/image-20220331210846540-20220331211152512.png)
   
4. 构建触发器-选择 轮询SCM

   ![image-20220331211326706](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/image-20220331211326706.png)
   
5. 构建-选择执行shell

   ```shell
   echo "building......";
   cat ./README.md
   ```

   

6. 点击保存



### 五、项目开始自动执行

1. 自动部署

   ![image-20220331211558241](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/image-20220331211558241.png)

   

2. 运行成功-查看输出的日志

   ![image-20220331211640964](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/image-20220331211640964.png)

