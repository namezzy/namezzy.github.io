---
 layout: post
title: Dubbo+Zookeeper+Dubbo-admin以及整合SpingBoot
subtitle: 微服务
date: 2022-07-08
author: Levi
header-img: img/dubbo/ms.jpeg
catalog: true
top: false
tags:
  - Dubbo
  - Zookeeper
  - Dubbo-admin
  - Springboot
  - Micrervice
  
---



### 一、Dubbo

Dubbo是 阿里巴巴公司开源的一个高性能优秀的服务框架，使得应用可通过高性能的 RPC 实现服务的输出和输入功能，可以和 Spring框架无缝集成。

Dubbo是一款高性能、轻量级的开源Java RPC框架，它提供了三大核心能力：面向接口的远程方法调用，智能容错和负载均衡，以及服务自动注册和发现。

[Dubbo官网](https://dubbo.apache.org/en/)



### 1、Dubbo是什么

Dubbo是一个分布式服务框架，致力于提供高性能和透明化的RPC远程服务调用方案，以及SOA服务治理方案。简单的说，dubbo就是个服务框架，如果没有分布式的需求，其实是不需要用的，只有在分布式的时候，才有dubbo这样的分布式服务框架的需求，并且本质上是用于服务调用的。说白了就是个远程服务调用的分布式框架。



**其核心部分包含**：

- 远程通讯：提供对多种基于长连接的NIO框架抽象封装，包括多种线程模型，序列化，以及“请求-响应”模式的信息交换方式。
-  集群容错：提供基于接口方法的透明远程过程调用，包括多协议支持，以及软负载均衡，失败容错，地址路由，动态配置等集群支持。
- 自动发现：基于注册中心目录服务，使服务消费方能动态的查找服务提供方，使地址透明，使服务提供方可以平滑增加或减少机器。

### 2、Dubbo能做什么

- 透明化的远程方法调用，就像调用本地方法一样调用远程方法，只需简单配置，没有任何API侵入。    
- 软负载均衡及容错机制，可在内网替代F5等硬件负载均衡器，降低成本，减少单点。
- 服务自动注册与发现，不再需要写死服务提供方地址，注册中心基于接口名查询服务提供者的IP地址，并且能够平滑添加或删除服务提供者

​	**主要核心**

	1. Remoting: 网络通信框架，实现了 sync-over-async 和request-response 消息机制
	1. RPC: 一个远程过程调用的抽象，支持负载均衡、容灾和集群功能
	1. Registry: 服务目录框架用于服务的注册和服务事件发布和订阅

>  比如可访问互联网服务天气接口：
>
> 中国天气网地址：http://www.weather.com.cn
>
> 请求地1.1.1 互联网服务址：http://www.weather.com.cn/data/sk/101110101.html
>
> 101010100=北京
>
> 101020100=上海
>
> 101210101=杭州
>
> 京东万象： https://wx.jcloud.com/api		



### 3、Dubbo服务的实现原理

Dubbo 的底层实现是动态代理， 由 Dubbo 框架创建远程服务（接口）对象的代理对象， 通过代理对象调用远程方法。

![image-20220708085845041](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/image-20220708085845041.png)



### 4、配置

下载链接：[zookeeper](https://zookeeper.apache.org/releases.html)

1. 下载之后解压缩得到以下目录

   ![image-20220708113021154](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/image-20220708113021154.png)

 



2. 在confi目录创建配置文件-zoo.cfg

> 只需要把原目录中的zoo_sample.cfg 复制一份重命名为zoo.cfg

​	![image-20220708113241500](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/image-20220708113241500.png)



```shell
# The number of milliseconds of each tick
tickTime=2000

# 自定义启动的服务端口,默认是8080 因为会和dubbo-admin的启动端口8080冲突，所以可以按照自己的需求进行修改
admin.serverPort=8888
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
# 数据存放的目录
dataDir=/Users/levi.zhao/Downloads/apache-zookeeper-3.8.0-bin/data
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# https://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1

## Metrics Providers
#
# https://prometheus.io Metrics Exporter
#metricsProvider.className=org.apache.zookeeper.metrics.prometheus.PrometheusMetricsProvider
#metricsProvider.httpHost=0.0.0.0
#metricsProvider.httpPort=7000
#metricsProvider.exportJvmInfo=true


```



### 5、部署

1. 打开bin目录

![image-20220708114223852](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/image-20220708114223852.png)



2. 分别启动zkServer.sh(服务端)和zkCli.sh(客户端)

   ```bash
   #启动zkServer.sh
   ./zkServer.sh start
   
   #查看zkServer的运行状态
   ./zkServer.sh status
   
   #停止运行zkServer
   ./zkServer.sh stop
   
   ```

   

   ![image-20220708115206814](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/image-20220708115206814.png)

   

   ![image-20220708115255457](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/image-20220708115255457.png)

   

   ```bash
   #启动zkCli.sh
   ./zkCli.sh
   ```

   ​	![image-20220708115452892](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/image-20220708115452892.png)



3. dubbo的一些命令用法

   ```bash
   #ls /: 列出zookeeper根下保存的所有节点
   [zk: localhost:2181(CONNECTED) 0] ls /
   [dubbo, services, zookeeper]
   [zk: localhost:2181(CONNECTED) 1]
   
   # create -e /levi 123: 创建一个levi节点，值为123
   [zk: localhost:2181(CONNECTED) 3] create -e /levi 123
   Created /levi
   
   #get /levi: 获取/levi节点的值
   [zk: localhost:2181(CONNECTED) 4] get /levi
   123
   ```
   



### 6、Dubbo-Admin安装

dubbo-admin项目下载链接：[appache/dubbo](https://github.com/apache/dubbo-admin/tree/master)

1. 环境准备

   dubbo-admin 是一个前后端分离的项目。前端使用vue，后端使用springboot，安装 dubbo-admin 其实就是部署该项目。我们将dubbo-admin安装到开发环境上。要保证开发环境有jdk，maven，nodejs

2. 下载Dubbo-Admin

   ```bash
   git clone https://github.com/apache/dubbo.git
   ```

3. 下载解压缩之后进入…/dubbo-admin/dubbo-admin-server/src/main/resources看一下配置文件

   ```properties
   #
   # Licensed to the Apache Software Foundation (ASF) under one or more
   # contributor license agreements.  See the NOTICE file distributed with
   # this work for additional information regarding copyright ownership.
   # The ASF licenses this file to You under the Apache License, Version 2.0
   # (the "License"); you may not use this file except in compliance with
   # the License.  You may obtain a copy of the License at
   #
   #     http://www.apache.org/licenses/LICENSE-2.0
   #
   # Unless required by applicable law or agreed to in writing, software
   # distributed under the License is distributed on an "AS IS" BASIS,
   # WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   # See the License for the specific language governing permissions and
   # limitations under the License.
   #
   
   # centers in dubbo2.7, if you want to add parameters, please add them to the url
   admin.registry.address=zookeeper://127.0.0.1:2181
   admin.config-center=zookeeper://127.0.0.1:2181
   admin.metadata-report.address=zookeeper://127.0.0.1:2181
   
   # nacos config, add parameters to url like username=nacos&password=nacos
   #admin.registry.address=nacos://127.0.0.1:8848?group=DEFAULT_GROUP&namespace=public
   #admin.config-center=nacos://127.0.0.1:8848?group=dubbo
   #admin.metadata-report.address=nacos://127.0.0.1:8848?group=dubbo
   
   #group (Deprecated it is recommended to use URL to add parameters,will be removed in the future)
   #admin.registry.group=dubbo
   #admin.config-center.group=dubbo
   #admin.metadata-report.group=dubbo
   
   #namespace used by nacos. (Deprecated it is recommended to use URL to add parameters,will be removed in the future)
   #admin.registry.namespace=public
   #admin.config-center.namespace=public
   #admin.metadata-report.namespace=public
   
   admin.root.user.name=root
   admin.root.user.password=root
   ser
   
   #session timeout, default is one hour
   admin.check.sessionTimeoutMilli=3600000
   
   
   # apollo config
   # admin.config-center = apollo://localhost:8070?token=e16e5cd903fd0c97a116c873b448544b9d086de9&app.id=test&env=dev&cluster=default&namespace=dubbo
   
   # (Deprecated it is recommended to use URL to add parameters,will be removed in the future)
   #admin.apollo.token=e16e5cd903fd0c97a116c873b448544b9d086de9
   #admin.apollo.appId=test
   #admin.apollo.env=dev
   #admin.apollo.cluster=default
   #admin.apollo.namespace=dubbo
   
   #compress
   server.compression.enabled=true
   server.compression.mime-types=text/css,text/javascript,application/javascript
   server.compression.min-response-size=10240
   
   #token timeout, default is one hour
   admin.check.tokenTimeoutMilli=3600000
   #Jwt signingKey
   admin.check.signSecret=86295dd0c4ef69a1036b0b0c15158d77
   
   #dubbo config
   dubbo.application.name=dubbo-admin
   dubbo.registry.address=${admin.registry.address}
   
   # mysql
   #spring.datasource.driver-class-name=com.mysql.jdbc.Driver
   #spring.datasource.url=jdbc:mysql://localhost:3306/dubbo-admin?characterEncoding=utf8&connectTimeout=1000&socketTimeout=10000&autoReconnect=true
   #spring.datasource.username=root
   #spring.datasource.password=mysql
   
   # h2
   spring.datasource.url=jdbc:h2:mem:~/dubbo-admin;MODE=MYSQL;
   spring.datasource.username=sa
   spring.datasource.password=
   
   # id generate type
   mybatis-plus.global-config.db-config.id-type=none
   ```

4. 打包编译运行项目(也可以直接运行别人打包好的jar包)

   ```bash
   #Build
   mvn clean package -Dmaven.test.skip=true
   
   #cd dubbo-admin-distribution/target
   java -jar dubbo-admin-0.4.jar
   
   # 启动之后打开浏览器访问：http://localhost:8080
   # 登录的账户和密码都是root root
   ```


   ![image-20220708122321937](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/image-20220708122321937.png)



### 7、整合Springboot

1. 新建一个空的java项目：dubbo-zookeeper

2. 添加两个module分别为：provider-server和consumer-server

   ![image-20220708122904675](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/image-20220708122904675.png)

3. provider-server

   导入所用到依赖

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>
       <parent>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-parent</artifactId>
           <version>2.7.1</version>
           <relativePath/> <!-- lookup parent from repository -->
       </parent>
       <groupId>top.withlevi</groupId>
       <artifactId>provider-server</artifactId>
       <version>0.0.1-SNAPSHOT</version>
       <name>provider-server</name>
       <description>provider-server</description>
       <properties>
           <java.version>1.8</java.version>
       </properties>
       <dependencies>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
   
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-test</artifactId>
               <scope>test</scope>
           </dependency>
   
           <dependency>
               <groupId>org.apache.dubbo</groupId>
               <artifactId>dubbo-spring-boot-starter</artifactId>
               <version>2.7.8</version>
           </dependency>
   
           <!-- https://mvnrepository.com/artifact/org.apache.curator/curator-recipes -->
           <dependency>
               <groupId>org.apache.curator</groupId>
               <artifactId>curator-recipes</artifactId>
               <version>5.2.0</version>
           </dependency>
           <!-- https://mvnrepository.com/artifact/org.apache.curator/curator-framework -->
           <dependency>
               <groupId>org.apache.curator</groupId>
               <artifactId>curator-framework</artifactId>
               <version>5.2.0</version>
           </dependency>
       </dependencies>
   
       <build>
           <plugins>
               <plugin>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-maven-plugin</artifactId>
               </plugin>
           </plugins>
       </build>
   
   </project>
   
   ```


   编写application.yml文件

   ```yaml
   dubbo:
     registry:
       protocol: zookeeper
       address: zookeeper://localhost:2181
     protocol:
       name: dubbo
       port: 20881
     scan:
       base-packages: top.withlevi.providerserver.service
   
   
   spring:
     application:
       name: provider-server
   server:
     port: 8081
   
   ```

   编写服务类

   TicketService

   ```java
   package top.withlevi.providerserver.service;
   
   /**
    * @author Levi.Zhao
    * @version 1.0
    * @date 2022/7/5 11:19 PM
    */
   public interface TicketService {
   
       public String getTicket();
   }
   
   ```

   TicketServiceImpl 

   ```java
   package top.withlevi.providerserver.service;
   
   import org.apache.dubbo.config.annotation.DubboService;
   import org.springframework.stereotype.Component;
   import top.withlevi.providerserver.service.TicketService;
   
   /**
    * @author Levi.Zhao
    * @version 1.0
    * @date 2022/7/5 11:20 PM
    */
   
   @DubboService
   public class TicketServiceImpl implements TicketService {
       @Override
       public String getTicket() {
           return "卖票服务";
       }
   }
   
   ```


   启动-ProviderServerApplication之后在Dubbo Admin 可以查看运行服务

   ![image-20220708123507617](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/image-20220708123507617.png)

4. consumer-server

   导入所需要的依赖

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>
       <parent>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-parent</artifactId>
           <version>2.7.1</version>
           <relativePath/> <!-- lookup parent from repository -->
       </parent>
       <groupId>top.withlevi</groupId>
       <artifactId>consumer-server</artifactId>
       <version>0.0.1-SNAPSHOT</version>
       <name>consumer-server</name>
       <description>consumer-server</description>
       <properties>
           <java.version>1.8</java.version>
       </properties>
       <dependencies>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
   
           <dependency>
               <groupId>org.apache.dubbo</groupId>
               <artifactId>dubbo-spring-boot-starter</artifactId>
               <version>2.7.8</version>
           </dependency>
   
           <!-- https://mvnrepository.com/artifact/org.apache.curator/curator-recipes -->
           <dependency>
               <groupId>org.apache.curator</groupId>
               <artifactId>curator-recipes</artifactId>
               <version>5.2.0</version>
           </dependency>
           <!-- https://mvnrepository.com/artifact/org.apache.curator/curator-framework -->
           <dependency>
               <groupId>org.apache.curator</groupId>
               <artifactId>curator-framework</artifactId>
               <version>5.2.0</version>
           </dependency>
   
           <dependency>
               <groupId>top.withlevi</groupId>
               <artifactId>provider-server</artifactId>
               <version>0.0.1-SNAPSHOT</version>
           </dependency>
   
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-test</artifactId>
               <scope>test</scope>
           </dependency>
         
         <--这里要注意如果想使用其他服务的类，需要在这里设置pom坐标-->
           <dependency>
               <groupId>top.withlevi</groupId>
               <artifactId>provider-server</artifactId>
               <version>0.0.1-SNAPSHOT</version>
               <scope>compile</scope>
           </dependency>
       </dependencies>
   
       <build>
           <plugins>
               <plugin>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-maven-plugin</artifactId>
               </plugin>
           </plugins>
       </build>
   
   </project>
   
   ```

   application.yml

   ```yaml
   dubbo:
     registry:
       protocol: zookeeper
       address: zookeeper://localhost:2181
     protocol:
       name: dubbo
       port: 20882
   
   
   
   spring:
     application:
       name: consumer-server
   
   server:
     port: 8082
   ```

   编写UserService类

   UserService

   ```java
   package top.withlevi.consumerserver.service;
   
   import org.apache.dubbo.config.annotation.DubboReference;
   import org.apache.dubbo.config.annotation.DubboService;
   import org.apache.dubbo.config.annotation.Reference;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.stereotype.Component;
   import org.springframework.stereotype.Service;
   import top.withlevi.providerserver.service.TicketService;
   
   @Service
   public class UserService {
   
       @DubboReference
       TicketService ticketService;
   
       public void buyTicket(){
           System.out.println(ticketService.getTicket());
       }
   }
   
   ```

   测试类

   ```java
   package top.withlevi.consumerserver;
   
   import org.junit.jupiter.api.Test;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.test.context.SpringBootTest;
   import top.withlevi.consumerserver.service.UserService;
   
   import javax.annotation.Resource;
   
   @SpringBootTest
   class ConsumerServerApplicationTests {
   
       @Autowired
       UserService userService;
   
       @Test
       void contextLoads() {
           userService.buyTicket();
       }
   
   }
   
   ```

   启动ConsumerServerApplication 之后在Dubbo Admin 可以查看运行服务

   ![image-20220708124318609](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/image-20220708124318609.png)

   

   

















