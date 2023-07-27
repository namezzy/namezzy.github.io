---
layout: post
title: Spring Data JPA
subtitle: JPA
date: 2023-06-13
author: Levi
header-img: img/jpa/database.png
catalog: true
top: false
tags:
  - JPA
  - SQL
  - Springboot
---



![image-20230613164409165](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/202306131644313.png)





## 持久层框架：JPA

### 认识SpringDataJPA

JPA（Java Persistence API）和JDBC类似，也是官方定义的一组接口，但是它相比传统的JDBC，它是为了实现ORM而生的，即Object-Relationl Mapping，它的作用是在关系型数据库和对象之间形成一个映射，这样，我们在具体的操作数据库的时候，就不需要再去和复杂的SQL语句打交道，只要像平时操作对象一样操作它就可以了。

在之前，我们使用JDBC或是Mybatis来操作数据，通过直接编写对应的SQL语句来实现数据访问，但是我们发现实际上我们在Java中大部分操作数据库的情况都是读取数据并封装为一个实体类，因此，为什么不直接将实体类直接对应到一个数据库表呢？也就是说，一张表里面有什么属性，那么我们的对象就有什么属性，所有属性跟数据库里面的字段一一对应，而读取数据时，只需要读取一行的数据并封装为我们定义好的实体类既可以，而具体的SQL语句执行，完全可以交给框架根据我们定义的映射关系去生成，不再由我们去编写，因为这些SQL实际上都是千篇一律的。

而实现JPA规范的框架一般最常用的就是`Hibernate`，它是一个重量级框架，学习难度相比Mybatis也更高一些，而SpringDataJPA也是采用Hibernate框架作为底层实现，并对其加以封装。

官网：https://spring.io/projects/spring-data-jpa



### 使用JPA

只需要导入stater依赖即可

```xml
 <dependencies>
     	<--jpa依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
		
            <--数据库依赖-->
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <scope>runtime</scope>
        </dependency>
                
        <--lombok为了省去写setter和getter-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
            <--测试-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

接着我们可以直接创建一个类，比如账户类，我们只需要把一个账号对应的属性全部定义好即可：

```java
@Data
public class Account {
    int id;
    String username;
    String password;
}
```

接着，我们可以通过注解形式，在属性上添加数据库映射关系，这样就能够让JPA知道我们的实体类对应的数据库表是什么样子

```java

@Data
@Entity // 表示这个类是一个实体
@Table(name = "accounts") //  对应的数据库中创建的表名称
public class Account {

    @Column(name = "id") // 对应表中id这一列
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Id //  此为主键属性
    private int id;

    @Column(name = "username", nullable = false, unique = true)
    private String username;

    @Column(name = "password", nullable = false, unique = true)
    private String password;

}

```

配置文件

```yaml
spring:
  datasource:
    url: jdbc:mysql://bj-cynosdbmysql-grp-i36rotgc.sql.com/databases
    username: user
    password: password
    driver-class-name: com.mysql.cj.jdbc.Driver
  jpa:
    properties:
      hibernate:
        hbm2ddl:
        # 配置为自动创建 create 、create-drop、update、validate
          auto: update
        dialect: org.hibernate.dialect.MySQL5InnoDBDialect
        # 格式化SQL语句
      format_sql: true
    # 开启SQL语句执行日志信息  
    show-sql: true

```

`ddl-auto`属性用于设置自动表定义，可以实现自动在数据库中为我们创建一个表，表的结构会根据我们定义的实体类决定，它有4种

* create 启动时删数据库中的表，然后创建，退出时不删除数据表 
* create-drop 启动时删数据库中的表，然后创建，退出时删除数据表 如果表不存在报错 
* update 如果启动时表格式不一致则更新表，原有数据保留 
* validate 项目启动表结构进行校验 如果不一致则报错

我们可以在日志中发现，在启动时执行了如下SQL语句：

```shell
Hibernate: create table users (id integer not null auto_increment, password varchar(255), username varchar(255), primary key (id)) engine=InnoDB
```

而我们的数据库中对应的表已经创建好了。

我们接着来看如何访问我们的表，我们需要创建一个Repository实现类：

```java
@Repository
public interface AccountRepository extends JpaRepository<Account, Integer> {

}

```

注意JpaRepository有两个泛型，前者是具体操作的对象实体，也就是对应的表，后者是ID的类型，接口中已经定义了比较常用的数据库操作。编写接口继承即可，我们可以直接注入此接口获得实现类：

```java
@SpringBootTest
class JpaTestApplicationTests {

    @Resource
    AccountRepository repository;

    @Test
    void contextLoads() {
      	//直接根据ID查找
        repository.findById(1).ifPresent(System.out::println);
    }

}
```

运行后，成功得到查询结果。我们接着来测试增删操作：

```java
@Test
void addAccount(){
    Account account = new Account();
    account.setUsername("Admin");
    account.setPassword("123456");
    account = repository.save(account);  //返回的结果会包含自动生成的主键值
    System.out.println("插入时，自动生成的主键ID为："+account.getId());
}
```

```java
@Test
void deleteAccount(){
    repository.deleteById(2);   //根据ID删除对应记录
}
```

```java
@Test
void pageAccount() {
    repository.findAll(PageRequest.of(0, 1)).forEach(System.out::println);  //直接分页
}
```

我们发现，使用了JPA之后，整个项目的代码中没有出现任何的SQL语句，可以说是非常方便了，JPA依靠我们提供的注解信息自动完成了所有信息的映射和关联。

相比Mybatis，JPA几乎就是一个全自动的ORM框架，而Mybatis则顶多算是半自动ORM框架。

### 方法名称拼接自定义SQL

的条件判断名称：

| `Distinct`             | `findDistinctByLastnameAndFirstname`                         | `select distinct … where x.lastname = ?1 and x.firstname = ?2` |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `And`                  | `findByLastnameAndFirstname`                                 | `… where x.lastname = ?1 and x.firstname = ?2`               |
| `Or`                   | `findByLastnameOrFirstname`                                  | `… where x.lastname = ?1 or x.firstname = ?2`                |
| `Is`，`Equals`         | `findByFirstname`,`findByFirstnameIs`,`findByFirstnameEquals` | `… where x.firstname = ?1`                                   |
| `Between`              | `findByStartDateBetween`                                     | `… where x.startDate between ?1 and ?2`                      |
| `LessThan`             | `findByAgeLessThan`                                          | `… where x.age < ?1`                                         |
| `LessThanEqual`        | `findByAgeLessThanEqual`                                     | `… where x.age <= ?1`                                        |
| `GreaterThan`          | `findByAgeGreaterThan`                                       | `… where x.age > ?1`                                         |
| `GreaterThanEqual`     | `findByAgeGreaterThanEqual`                                  | `… where x.age >= ?1`                                        |
| `After`                | `findByStartDateAfter`                                       | `… where x.startDate > ?1`                                   |
| `Before`               | `findByStartDateBefore`                                      | `… where x.startDate < ?1`                                   |
| `IsNull`，`Null`       | `findByAge(Is)Null`                                          | `… where x.age is null`                                      |
| `IsNotNull`，`NotNull` | `findByAge(Is)NotNull`                                       | `… where x.age not null`                                     |
| `Like`                 | `findByFirstnameLike`                                        | `… where x.firstname like ?1`                                |
| `NotLike`              | `findByFirstnameNotLike`                                     | `… where x.firstname not like ?1`                            |
| `StartingWith`         | `findByFirstnameStartingWith`                                | `… where x.firstname like ?1`（参数与附加`%`绑定）           |
| `EndingWith`           | `findByFirstnameEndingWith`                                  | `… where x.firstname like ?1`（参数与前缀`%`绑定）           |
| `Containing`           | `findByFirstnameContaining`                                  | `… where x.firstname like ?1`（参数绑定以`%`包装）           |
| `OrderBy`              | `findByAgeOrderByLastnameDesc`                               | `… where x.age = ?1 order by x.lastname desc`                |
| `Not`                  | `findByLastnameNot`                                          | `… where x.lastname <> ?1`                                   |
| `In`                   | `findByAgeIn(Collection<Age> ages)`                          | `… where x.age in ?1`                                        |
| `NotIn`                | `findByAgeNotIn(Collection<Age> ages)`                       | `… where x.age not in ?1`                                    |
| `True`                 | `findByActiveTrue()`                                         | `… where x.active = true`                                    |
| `False`                | `findByActiveFalse()`                                        | `… where x.active = false`                                   |
| `IgnoreCase`           | `findByFirstnameIgnoreCase`                                  | `… where UPPER(x.firstname) = UPPER(?1)                      |

比如想要实现根据用户名模糊匹配查找用户：

```java
@Repository
public interface AccountRepository extends JpaRepository<Account, Integer> {
		//按照表中的规则进行名称拼接，不用刻意去记，IDEA会有提示
    List<Account> findAllByUsernameLike(String str);
}
```

```java
@Test
void test() {
    repository.findAllByUsernameLike("%T%").forEach(System.out::println);
}
```

又比如想同时根据用户名和ID一起查询：

```java
@Repository
public interface AccountRepository extends JpaRepository<Account, Integer> {

    Account findByIdAndUsername(int id, String username);
  //可以使用Optional类进行包装，Optional<Account> findByIdAndUsername(int id, String username);
    
    List<Account> findAllByUsernameLike(String str);
}
```

```java
@Test
void test() {
    System.out.println(repository.findByIdAndUsername(1, "Test"));
}
```

比如想判断数据库中是否存在某个ID的用户：

```java
@Repository
public interface AccountRepository extends JpaRepository<Account, Integer> {

    Account findByIdAndUsername(int id, String username);

    List<Account> findAllByUsernameLike(String str);

    boolean existsAccountById(int id);
}
```

```java
@Test
void test() {
    System.out.println(repository.existsAccountByUsername("Test"));
}
```

注意自定义条件操作的方法名称一定要遵循规则，不然会出现异常：

```shell
Caused by: org.springframework.data.repository.query.QueryCreationException: Could not create query for public abstract  ...
```



### 关联查询

在实际开发中，比较常见的场景还有关联查询，也就是我们会在表中添加一个外键字段，而此外键字段又指向了另一个表中的数据，当我们查询数据时，可能会需要将关联数据也一并获取，比如我们想要查询某个用户的详细信息，一般用户简略信息会单独存放一个表，而用户详细信息会单独存放在另一个表中。当然，除了用户详细信息之外，可能在某些电商平台还会有用户的购买记录、用户的购物车，交流社区中的用户帖子、用户评论等，这些都是需要根据用户信息进行关联查询的内容。

![img](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/202306131718727.jpeg)

我们知道，在JPA中，每张表实际上就是一个实体类的映射，而表之间的关联关系，也可以看作对象之间的依赖关系，比如用户表中包含了用户详细信息的ID字段作为外键，那么实际上就是用户表实体中包括了用户详细信息实体对象：

```java
@Data
@Entity
@Table(name = "account_detail")
public class AccountDetail {

    @Column(name = "id")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Id
    int id;

    @Column(name = "address")
    String address;

    @Column(name = "email")
    String email;

    @Column(name = "phone")
    String phone;

    @Column(name = "real_name")
    String realName;
}
```

而用户信息和用户详细信息之间形成了一对一的关系，那么这时我们就可以直接在类中指定这种关系：

```java
@Data
@Entity
@Table(name = "accounts")
public class Account {

    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
    @Id
    int id;

    @Column(name = "username")
    String username;

    @Column(name = "password")
    String password;

    @JoinColumn(name = "detail_id")   //指定存储外键的字段名称
    @OneToOne    //声明为一对一关系
    AccountDetail detail;
}
```



在修改实体类信息后，我们发现在启动时也进行了更新，日志如下：

```shell
Hibernate: alter table users add column detail_id integer
Hibernate: create table users_detail (id integer not null auto_increment, address varchar(255), email varchar(255), phone varchar(255), real_name varchar(255), primary key (id)) engine=InnoDB
Hibernate: alter table users add constraint FK7gb021edkxf3mdv5bs75ni6jd foreign key (detail_id) references users_detail (id)
```

是不是感觉非常方便！都懒得去手动改表结构了。

接着往用户详细信息中添加一些数据，一会我们可以直接进行查询：

```java
@Test
void pageAccount() {
    repository.findById(1).ifPresent(System.out::println);
}
```

查询后，可以发现，得到如下结果：

```shell
Hibernate: select account0_.id as id1_0_0_, account0_.detail_id as detail_i4_0_0_, account0_.password as password2_0_0_, account0_.username as username3_0_0_, accountdet1_.id as id1_1_1_, accountdet1_.address as address2_1_1_, accountdet1_.email as email3_1_1_, accountdet1_.phone as phone4_1_1_, accountdet1_.real_name as real_nam5_1_1_ from users account0_ left outer join users_detail accountdet1_ on account0_.detail_id=accountdet1_.id where account0_.id=?
Account(id=1, username=Test, password=123456, detail=AccountDetail(id=1, address=北京市西城区, email=xxxxx@qq.com, phone=1234567890, realName=PDD))
```

也就是，在建立关系之后，我们查询Account对象时，会自动将关联数据的结果也一并进行查询。

那要是我们只想要Account的数据，不想要用户详细信息数据怎么办呢？我希望在我要用的时候再获取详细信息，这样可以节省一些网络开销，我们可以设置懒加载，这样只有在需要时才会向数据库获取：

```java
package top.withlevi.model;

import lombok.Data;
import org.hibernate.annotations.Fetch;

import javax.persistence.*;
import java.util.List;

/**
 * Created by Levi Zhao on 6/12/2023 9:15 AM
 *
 * @Author Levi
 */

@Data
@Entity
@Table(name = "accounts")
public class Account {

    @Column(name = "id")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Id
    private int id;

    @Column(name = "username", nullable = false, unique = true)
    private String username;

    @Column(name = "password", nullable = false, unique = true)
    private String password;

	// 将获取类型改为懒加载
    @OneToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    @JoinColumn(name = "detail_id")
    AccountDetails details;

}

```

接着我们测试一下：

```java
@Transactional   //懒加载属性需要在事务环境下获取，因为repository方法调用完后Session会立即关闭
@Test
void pageAccount() {
    repository.findById(1).ifPresent(account -> {
        System.out.println(account.getUsername());   //获取用户名
        System.out.println(account.getDetail());  //获取详细信息（懒加载）
    });
}
```

控制台输出：

```shell
Hibernate: select account0_.id as id1_0_0_, account0_.detail_id as detail_i4_0_0_, account0_.password as password2_0_0_, account0_.username as username3_0_0_ from users account0_ where account0_.id=?
Test
Hibernate: select accountdet0_.id as id1_1_0_, accountdet0_.address as address2_1_0_, accountdet0_.email as email3_1_0_, accountdet0_.phone as phone4_1_0_, accountdet0_.real_name as real_nam5_1_0_ from users_detail accountdet0_ where accountdet0_.id=?
AccountDetail(id=1, address=北京市西城区, email=8371289@qq.com, phone=1234567890, realName=PDD)
```

可以看到，获取用户名之前，并没有去查询用户的详细信息，而是当我们获取详细信息时才进行查询并返回AccountDetail对象。

那么我们是否也可以在添加数据时，利用实体类之间的关联信息，一次性添加两张表的数据呢？可以，但是我们需要稍微修改一下级联关联操作设定：

```java
package top.withlevi.model;

import lombok.Data;
import org.hibernate.annotations.Fetch;

import javax.persistence.*;
import java.util.List;

/**
 * Created by Levi Zhao on 6/12/2023 9:15 AM
 *
 * @Author Levi
 */

@Data
@Entity
@Table(name = "accounts")
public class Account {

    @Column(name = "id")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Id
    private int id;

    @Column(name = "username", nullable = false, unique = true)
    private String username;

    @Column(name = "password", nullable = false, unique = true)
    private String password;

	//设置关联操作为ALL
    @OneToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    @JoinColumn(name = "detail_id")
    AccountDetails details;


}

```

* ALL：所有操作都进行关联操作
* PERSIST：插入操作时才进行关联操作
* REMOVE：删除操作时才进行关联操作
* MERGE：修改操作时才进行关联操作

可以多个并存，接着我们来进行一下测试：

```java
@Test
void addAccount(){
    Account account = new Account();
    account.setUsername("root");
    account.setPassword("root@123456");
    AccountDetail detail = new AccountDetail();
    detail.setAddress("北京市丰台区");
    detail.setPhone("1234567890");
    detail.setEmail("73281937@qq.com");
    detail.setRealName("小学生");
  	account.setDetail(detail);
    account = repository.save(account);
    System.out.println("插入时，自动生成的主键ID为："+account.getId()+"，外键ID为："+account.getDetail().getId());
}
```

可以看到日志结果：

```shell
Hibernate: insert into users_detail (address, email, phone, real_name) values (?, ?, ?, ?)
Hibernate: insert into users (detail_id, password, username) values (?, ?, ?)
插入时，自动生成的主键ID为：6，外键ID为：3
```

结束后会发现数据库中两张表都同时存在数据。

接着我们来看一对多关联，比如每个用户的成绩信息：

```java
package top.withlevi.model;

import lombok.Data;
import org.hibernate.annotations.Fetch;

import javax.persistence.*;
import java.util.List;

/**
 * Created by Levi Zhao on 6/12/2023 9:15 AM
 *
 * @Author Levi
 */

@Data
@Entity
@Table(name = "accounts")
public class Account {

    @Column(name = "id")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Id
    private int id;

    @Column(name = "username", nullable = false, unique = true)
    private String username;

    @Column(name = "password", nullable = false, unique = true)
    private String password;


    @OneToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    @JoinColumn(name = "detail_id")
    AccountDetails details;

 	//注意这里的name指的是Score表中的uid字段对应的就是当前的主键，会将uid外键设置为当前的主键
   @JoinColumn(name = "uid") 
    //在移除Account时，一并移除所有的成绩信息，依然使用懒加载
   @OneToMany(fetch = FetchType.LAZY, cascade = CascadeType.REMOVE)  
    List<Score> scoreList;

}

```

```java
@Data
@Entity
@Table(name = "account_score")   //成绩表，注意只存成绩，不存学科信息，学科信息id做外键
public class Score {

    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
    @Id
    int id;

    @OneToOne   //一对一对应到学科上
    @JoinColumn(name = "cid")
    Subject subject;

    @Column(name = "socre")
    double score;

    @Column(name = "uid")
    int uid;
}
```

```java
@Data
@Entity
@Table(name = "subjects")   //学科信息表
public class Subject {

    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "cid")
    @Id
    int cid;

    @Column(name = "name")
    String name;

    @Column(name = "teacher")
    String teacher;

    @Column(name = "time")
    int time;
}
```

在数据库中填写相应数据，接着我们就可以查询用户的成绩信息了：

```java
@Transactional
@Test
void test() {
    repository.findById(1).ifPresent(account -> {
        account.getScoreList().forEach(System.out::println);
    });
}
```

成功得到用户所有的成绩信息，包括得分和学科信息。

同样的，我们还可以将对应成绩中的教师信息单独分出一张表存储，并建立多对一的关系，因为多门课程可能由同一个老师教授（千万别搞晕了，一定要理清楚关联关系，同时也是考验你的基础扎不扎实）：

```java
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "tid")   //存储教师ID的字段，和一对一是一样的，也会当前表中创个外键
Teacher teacher;
```

```java
@Data
@Entity
@Table(name = "teachers")
public class Teacher {

    @Column(name = "id")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Id
    int id;

    @Column(name = "name")
    String name;

    @Column(name = "sex")
    String sex;
}
```

最后我们再进行一下测试：

```java
@Transactional
@Test
void test() {
    repository.findById(3).ifPresent(account -> {
        account.getScoreList().forEach(score -> {
            System.out.println("课程名称："+score.getSubject().getName());
            System.out.println("得分："+score.getScore());
            System.out.println("任课教师："+score.getSubject().getTeacher().getName());
        });
    });
}
```

成功得到多对一的教师信息。

最后我们再来看最复杂的情况，现在我们一门课程可以由多个老师教授，而一个老师也可以教授多个课程，那么这种情况就是很明显的多对多场景，现在又该如何定义呢？我们可以像之前一样，插入一张中间表表示教授关系，这个表中专门存储哪个老师教哪个科目：

```java
@ManyToMany(fetch = FetchType.LAZY)   //多对多场景
@JoinTable(name = "teach_relation",     //多对多中间关联表
        joinColumns = @JoinColumn(name = "cid"),    //当前实体主键在关联表中的字段名称
        inverseJoinColumns = @JoinColumn(name = "tid")   //教师实体主键在关联表中的字段名称
)
List<Teacher> teacher;
```



### JPQL自定义SQL语句

虽然SpringDataJPA能够简化大部分数据获取场景，但是难免会有一些特殊的场景，需要使用复杂查询才能够去完成，这时你又会发现，如果要实现，只能用回Mybatis了，因为我们需要自己手动编写SQL语句，过度依赖SpringDataJPA会使得SQL语句不可控。

使用JPA，我们也可以像Mybatis那样，直接编写SQL语句，不过它是JPQL语言，与原生SQL语句很类似，但是它是面向对象的，当然我们也可以编写原生SQL语句。

比如我们要更新用户表中指定ID用户的密码：

```java
@Repository
public interface AccountRepository extends JpaRepository<Account, Integer> {

    @Transactional    //DML操作需要事务环境，可以不在这里声明，但是调用时一定要处于事务环境下
    @Modifying     //表示这是一个DML操作
    @Query("update Account set password = ?2 where id = ?1") //这里操作的是一个实体类对应的表，参数使用?代表，后面接第n个参数
    int updatePasswordById(int id, String newPassword);
}
```

```java
@Test
void updateAccount(){
    repository.updatePasswordById(1, "654321");
}
```

现在我想使用原生SQL来实现根据用户名称修改密码：

```java
@Transactional
@Modifying
@Query(value = "update users set password = :pwd where username = :name", nativeQuery = true) //使用原生SQL，和Mybatis一样，这里使用 :名称 表示参数，当然也可以继续用上面那种方式。
int updatePasswordByUsername(@Param("name") String username,   //我们可以使用@Param指定名称
                             @Param("pwd") String newPassword);
```

```java
@Test
void updateAccount(){
    repository.updatePasswordByUsername("Admin", "654321");
}
```

通过编写原生SQL，在一定程度上弥补了SQL不可控的问题。

虽然JPA能够为我们带来非常便捷的开发体验，但是正式因为太便捷了，保姆级的体验有时也会适得其反，可能项目开发到后期特别庞大时，就只能从底层SQL语句开始进行优化，而由于JPA尽可能地在屏蔽我们对SQL语句的编写，所以后期优化是个大问题，并且Hibernate相对于Mybatis来说，更加重量级。不过，在微服务的时代，单体项目一般不会太大，而JPA的劣势并没有太明显地体现出来。

有关Mybatis和JPA的对比，可以参考：https://blog.csdn.net/u010253246/article/details/105731204