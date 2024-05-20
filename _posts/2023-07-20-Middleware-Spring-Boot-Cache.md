---
layout: post
title: Middleware-Spring Boot Cache
subtitle: Message Cache
date: 2023-07-20
author: Levi
header-img: img/linux/post-bg-computer.png
catalog: true
top: false
tags:
  - Spring Boot
  - Message Cache
  - Middleware
  - Cache
  - Redis
  - Cached
  - Topic
 
---





# Spring Boot 中使用 Cache 缓存



我们知道绝大多数的网站/系统，最先遇到的一个性能瓶颈就是数据库，使用缓存做数据库的前置缓存，可以
非常有效地降低数据库的压力，从而提升整个系统的响应效率和并发量。

以往使用缓存时，通常创建好缓存工具类，使用时将对应的工具类注入，操作工具类在前端处理缓存的逻
辑。其实这种方式是低效的，大部分使用缓存的场景是基于数据库的缓存，这类缓存场景的逻辑往往是：如
果缓存中存在数据，就从缓存中读取，如果缓存中不存在数据或者数据失效，就再从数据库中读取。

为了实现这样的逻辑，往往需要在业务代码中写很多的逻辑判断，那么有没有通用的代码来实现这样的逻辑
呢？其实有，按照这个逻辑我们可以写一个工具类来实现，每次需要这样判断逻辑时调用工具类中的方法即
可，还有没有更更优雅的使用方式呢？答案是肯定的，如果我们把这种固定的逻辑使用 Java 注解来实现，每
次需要使用时只需要在对应的方法或者类上写上注解即可。

Spring 也看到了这样的使用场景，于是有了注释驱动的 Spring Cache。它的原理是 Spring Cache 利用了
Spring AOP 的动态代理技术，在项目启动的时候动态生成它的代理类，在代理类中实现了对应的逻辑。

Spring Cache 是在 Spring 3.1 中引入的基于注释（Annotation）的缓存（Cache）技术，它本质上不是一个
具体的缓存实现方案，而是一个对缓存使用的抽象，通过在既有代码中添加少量定义的各种 Annotation，即能够达到缓存方法的返回对象的效果。

**Spring 的缓存技术还具备相当的灵活性**，不仅能够使用 SpEL（Spring Expression Language）来定义缓存
的 key 和各种 condition，还提供了开箱即用的缓存临时存储方案，也支持和主流的专业缓存如 EHCache 集
成。

> SpEL（Spring Expression Language）是一个支持运行时查询和操作对象图的强大的表达式语言，其语法类似于统一 EL，但提供了额外特性，显式方法调用和基本字符串模板函数。

其特点总结如下：

- 通过少量的配置 Annotation 注释即可使得既有代码支持缓存；
- 支持开箱即用 Out-Of-The-Box，即不用安装和部署额外第三方组件即可使用缓存；
- 支持 Spring Express Language，能使用对象的任何属性或者方法来定义缓存的 key 和 condition；
- 支持 AspectJ，并通过其实现任何方法的缓存支持；
- 支持自定义 key 和自定义缓存管理者，具有相当的灵活性和扩展性



## Spring Boot 中 Cache 的使用

Spring Boot 提供了了非常简单的解决方案，这里给大家演示最核心的三个注解：@Cacheable、
@CacheEvict、@CachePut。spring-boot-starter-cache 是 Spring Boot 体系内提供使用 Spring Cache 的
Starter 包。

在开始使用这三个注解之前，来介绍一个新的组件 **spring-boot-starter-cache**。

```xml
<dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-cache</artifactId>
</dependency>

```

Spring-boot-starter-cache 是 Spring Boot 提供缓存支持的 starter 包，其会进行缓存的自动化配置和识别，
Spring Boot 为 Redis 自动配置了 RedisCacheConfiguration 等信息，spring-boot-starter-cache 中的注解也
主要是使用了 Spring Cache 提供的支持。

### @Cacheable

@Cacheable 用来声明方法是可缓存的，将结果存储到缓存中以便后续使用相同参数调用时不需执行实际的
方法，直接从缓存中取值。@Cacheable 可以标记在一个方法上，也可以标记在一个类上。当标记在一个方
法上时表示该方法是支持缓存的，当标记在一个类上时则表示该类所有的方法都是支持缓存的。

**实例演示**

1. **需要引入的库**

   ```xml
   
       <properties>
           <java.version>1.8</java.version>
       </properties>
       <dependencies>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-cache</artifactId>
           </dependency>
   
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-data-redis</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-data-jpa</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
   
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-devtools</artifactId>
               <scope>runtime</scope>
               <optional>true</optional>
           </dependency>
           <dependency>
               <groupId>com.mysql</groupId>
               <artifactId>mysql-connector-j</artifactId>
               <scope>runtime</scope>
           </dependency>
           <dependency>
               <groupId>org.projectlombok</groupId>
               <artifactId>lombok</artifactId>
               <optional>true</optional>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-test</artifactId>
               <scope>test</scope>
           </dependency>
       </dependencies>
   
     
   
   </project>
   
   ```

2. **编写配置YAML文件**

   ```yaml
   spring:
     datasource:
       url: jdbc:mysql://localhost:3306/session
       driver-class-name: com.mysql.cj.jdbc.Driver
       username: 2Dm5aK84pRpb8VC.root
       password: 5GFWAHLrD9OpGf3M
     jpa:
       properties:
         hibernate:
           hbm2ddl:
             auto: update
           dialect: org.hibernate.dialect.MySQL5InnoDBDialect
       show-sql: true
   
     redis:
       host: localhost
       port: 6380
       password: levizhao
       database: 2
   
   
   ```

3. **编写model**

   ```java
   package top.withlevi.model;
   
   import lombok.AllArgsConstructor;
   import lombok.Data;
   import lombok.NoArgsConstructor;
   import javax.persistence.*;
   import java.io.Serializable;
   import java.util.Date;
   
   /**
    * By JPA
    *
    * @Author Levi
    */
   
   @Data
   @NoArgsConstructor
   @AllArgsConstructor
   @Entity
   @Table(name = "user2")
   public class User implements Serializable {
   
       @Id
       @Column(name = "id")
       @GeneratedValue(strategy = GenerationType.IDENTITY)
       private Long id;
   
       @Column(name = "username",unique = true,nullable = false)
       private String username;
   
       @Column(name = "password",nullable = false)
       private String password;
   
       @Column(name = "email",nullable = false)
       private String email;
   
       @Column(name = "regtime",nullable = false)
       private Date regTime;
   
   }
   
   ```

4. **Repository**

   ```java
   package top.withlevi.repository;
   
   import org.springframework.data.jpa.repository.JpaRepository;
   import org.springframework.stereotype.Repository;
   import top.withlevi.model.User;
   
   /**
    * 
    *
    * @Author Levi
    */
   
   @Repository
   public interface UserRepository extends JpaRepository<User, Long> {
   
       User findByUsername(String username);
   
       User findByUsernameOrEmail(String username, String email);
   }
   
   ```

5. **Controller**

   ```java
   package top.withlevi.controller;
   
   import org.springframework.cache.annotation.CacheEvict;
   import org.springframework.cache.annotation.CachePut;
   import org.springframework.cache.annotation.Cacheable;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   import top.withlevi.model.User;
   import top.withlevi.repository.UserRepository;
   
   import javax.annotation.Resource;
   
   /**
    * Created by Levi Zhao on 7/19/2023 4:58 PM
    *
    * @Author Levi
    */
   
   @RestController
   public class HelloController {
   
       @Resource
       private UserRepository userRepository;
   
       @RequestMapping("/hello")
       @Cacheable(value = "helloCache")
       public String hello(String name) {
           System.out.println("没有走缓存");
           return "hello " + name;
       }
   
   }
   
   ```

   来测试一下，启动项目后访问网址 http://localhost:8080/hello?name=Levi，输出：没有走缓存！，再次访问
   网址 http://localhost:8080/hello?name=Levi，输出栏没有变化，说明这次没有走 hello() 这个方法，内容直接由缓存返回。

   **@Cacheable(value="helloCache")** 这个注释的意思是，当调用这个方法时，会从一个名叫 helloCache 的缓存中查询，如果没有，则执行实际的方法（也可是查询数据库），并将执行的结果存入缓存中，否则返回缓
   存中的对象。这里的缓存中的 key 就是参数 name，value 就是返回的 String 值.

   **@Cacheable 支持如下几个参数。**

   - value：缓存的名称。
   - key：缓存的 key，可以为空，如果指定要按照 SpEL 表达式编写；如果不指定，则缺省按照方法的所有
     参数进行组合。
   - condition：触发条件，只有满足条件的情况才会加入缓存，默认为空，既表示全部都加入缓存，支持
     SpEL。

   我们把上面的方法稍微改成这样：

   ```java
   @RequestMapping("/condition")
   @Cacheable(value = "condition", condition = "#name.length() <=4")
   public String condition(String name) {
           System.out.println("Condition-没有走缓存");
           return "Hello " + name;
       }
   ```

   启动后在浏览器器中输入网址 http://localhost:8080/condition?name=Levi，第一次输出栏输出：没有走缓存！再次执行⽆输出，表明已经走缓存。在浏览器器中输入网址 http://localhost:8080/condition?name=Levi123456，浏览器器执行多次仍然一直输出：没有走缓存！说明条件 condition 生效。

   结合数据库的使用来做测试：

   ```java
   @RequestMapping("/getUser")
   @Cacheable(value = "userCache", key = "#username", condition = "#username.length() >3")
   public User getUser(String username) {
           User user = userRepository.findByUsername(username);
           System.out.println("Query in database.");
           return user;
       }
   ```

   启动后在浏览器器中输⼊网址 http://localhost:8080/getUser?username=Lev。

   Console中输出:

   ```shell
   2023-07-27 17:03:05.912  WARN 16084 --- [nio-8080-exec-6] com.zaxxer.hikari.pool.PoolBase          : HikariPool-1 - Failed to validate connection com.mysql.cj.jdbc.ConnectionImpl@70e5c028 (No operations allowed after connection closed.). Possibly consider using a shorter maxLifetime value.
   2023-07-27 17:03:05.914  WARN 16084 --- [nio-8080-exec-6] com.zaxxer.hikari.pool.PoolBase          : HikariPool-1 - Failed to validate connection com.mysql.cj.jdbc.ConnectionImpl@126026e2 (No operations allowed after connection closed.). Possibly consider using a shorter maxLifetime value.
   Query in database.
   ```

   多次执行，仍然输出上面的结果，说明每次请求都执行了数据库操作，再输入
   http://localhost:8080/getUser?username=Levi 进行测试。只有第一次返回了上面的内容，再次执行输
   出栏没有变化，说明后面的请求都已经从缓存中拿取了数据。

   最后总结一下：当执行到一个被 @Cacheable 注解的方法时，Spring 首先检查 condition 条件是否满足，如
   果不满足，执行方法，返回；如果满足，在缓存空间中查找使用 key 存储的对象，如果找到，将找到的结果
   返回，如果没有找到执行方法，将方法的返回值以 key-value 对象的方式存入缓存中，然后方法返回。

   > 需要注意的是当一个支持缓存的方法在对象内部被调用时是不会触发缓存功能的。

### @CachePut

项目运行中会对数据库的信息进行更新，如果仍然使用 **@Cacheable** 就会导致数据库的信息和缓存的信息不
一致。在以往的项目中，我们一般更新完数据库后，再手动删除掉 Redis 中对应的缓存，以保证数据的一致
性。Spring 提供了了另外的一种解决方案，可以让我们以优雅的方式去更新缓存。

与 @Cacheable 不同的是使用 @CachePut 标注的方法在执行前，不会去检查缓存中是否存在之前执行过
的结果，而是每次都会执行该方法，并将执行结果以键值对的形式存入指定的缓存中。

以上面的方法为例，我们再来做一个测试：

```java
@RequestMapping("/getputUser")
@CachePut(value = "userCache", key = "#username")
public User getPutUser(String username) {
        User user = userRepository.findByUsername(username);
        System.out.println("Query in database.");
        return user;
    }
```

我们新增一个 getPutUsers 方法，value、key 设置和 getUsers 方法保持一致，使用 @CachePut。同时手动
在数据库插入一条 username为xiaomijiao 的用户数据。

```sql
INSERT INTO `user2` VALUES('3','xiaomijiao','xiaomijiao@2023','2023-07-27','xmj@gmail.com')
```

在浏览器中输入网址 http://localhost:8080/getUser?username=xiaomijiao，并没有返回⽤用户昵称为
xiaomijiao 的用户信息，再次输入网址 http://localhost:8080/getputUser?username=xiaomijiao 可以查看到
此用户的信息，再次输入网址 http://localhost:8080/getUser?username=xiaomijiao 就可以看到用户昵称为
xiaomijiao的信息了。

说明执行在方法上声明 @CachePut 会自动执行方法，并将结果存入缓存。
