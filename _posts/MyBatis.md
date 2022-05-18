```
<!--get User Count-->
<select id="getCount" resultType="int">
    select count(*) from u_user;
</select>
```

# 					MyBatis

### 一、MyBatis简介

#### 1、MyBatis历史

MyBatis最初是Apache的一个开源项目**iBatis**, 2010年6月这个项目由Apache Software Foundation迁 移到了Google Code。随着开发团队转投Google Code旗下， iBatis3.x正式更名为MyBatis。代码于 2013年11月迁移到Github。

iBatis一词来源于“internet”和“abatis”的组合，是一个基于Java的持久层框架。 iBatis提供的持久层框架 包括SQL Maps和Data Access Objects(DAO)。

#### 2、MyBatis特性

1.  MyBatis 是支持定制化 SQL、存储过程以及高级映射的优秀的持久层框架
2. MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集
3.  MyBatis可以使用简单的XML或注解用于配置和原始映射，将接口和Java的POJO(Plain Old Java Objects，普通的Java对象)映射成数据库中的记录
4. MyBatis 是一个 半自动的ORM(Object Relation Mapping)框架

#### 3、Mybatis下载

> [mybatis下载地址](https://github.com/mybatis/mybatis-3)

#### 4、和其它持久层技术对比

- JDBC
  - SQL 夹杂在Java代码中耦合度高，导致硬编码内伤
  - 维护不易且实际开发需求中 SQL 有变化，频繁修改的情况多见
  - 代码冗长，开发效率低
- Hibernate 和 JPA
  - 操作简便，开发效率高
  - 程序中的长难复杂 SQL 需要绕过框架
  - 内部自动生产的 SQL，不容易做特殊优化
  - 基于全映射的全自动框架，大量字段的 POJO 进行部分映射时比较困难。
  - 反射操作太多，导致数据库性能下降
- MyBatis
  - 轻量级，性能出色
  - SQL 和 Java 编码分开，功能边界清晰。Java代码专注业务、SQL语句专注数据
  - 开发效率稍逊于HIbernate，但是完全能够接受



### 二、搭建MyBatis

#### 1、开发环境

- IntelliJ IDEA
- 构建工具 Maven
- Mysql
- Mybatis

#### 2、创建maven工程

1. 打包方式

   ```xml-dtd
   <packaging>jar</packaging>
   
   ```

2. 引入依赖

   ```xml
    <dependencies>
           <!-- Mybatis核心 -->
           <dependency>
               <groupId>org.mybatis</groupId>
               <artifactId>mybatis</artifactId>
               <version>3.5.7</version>
           </dependency>
           <!-- junit测试 -->
           <dependency>
               <groupId>junit</groupId>
               <artifactId>junit</artifactId>
               <version>4.12</version>
               <scope>test</scope>
           </dependency>
           <!-- MySQL驱动 -->
           <dependency>
               <groupId>mysql</groupId>
               <artifactId>mysql-connector-java</artifactId>
               <version>5.1.3</version>
           </dependency>
   
           <!-- log4j日志 -->
           <dependency>
               <groupId>log4j</groupId>
               <artifactId>log4j</artifactId>
               <version>1.2.17</version>
           </dependency>
       </dependencies>
   ```



#### 3、创建Mybatis的核心配置文件

​	习惯上命名为**mybatis-config.xml**，这个文件名仅仅只是建议，并非强制要求。将来整合Spring之后，这个配置文件可以省略，所以大家操作时可以直接复制、粘贴。 核心配置文件主要用于配置连接数据库的环境以及MyBatis的全局配置信息

**核心配置文件存放的位置是src/main/resources目录下**

```xml
<?xml version="1.0" encoding="UTF-8" ?> <!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--设置连接数据库的环境--> <environments default="development">
    <environment id="development">
        <transactionManager type="JDBC"/>
        <dataSource type="POOLED">
            <property name="driver" value="com.mysql.jdbc.Driver"/>
            <property name="url"
                      value="jdbc:mysql://localhost:3306/mybatis"/>
            <property name="username" value="root"/>
            <property name="password" value="root123456"/>
        </dataSource>
    </environment>
</environments>
    <!--引入映射文件-->
    <mappers>
        <mapper resource="UserMappers/UserMapper.xml"/>
    </mappers>
</configuration>
```



#### 4、创建mapper接口

> MyBatis中的mapper接口相当于以前的dao。但是区别在于mapper仅仅是接口，我们不需要提供实现类。

```java
package top.withlevi.mybatis.mapper;

import org.apache.ibatis.annotations.Param;
import top.withlevi.mybatis.pojo.User;
import java.util.List;

public interface UserMapper {

    /**
     * Add User
     * @return int
     */

    int insertUser();

    /**
     * Delete User
     * @return int
     */
    int deleteUser();

    /**
     * Update User
     * @return int
     */
    int updateUser();

    /**
     * getUserById
     * @return user
     */
    User getUserById();

    /**
     * getUserList
     * @return List
     */
    List<User> getUserList();


    User getUserById(@Param("id") int id);
}

```



#### 5、创建MyBatis的映射文件

相关概念:**ORM**(**O**bject **R**elationship **M**apping)对象关系映射。

- 对象：Java的实体类对象
- 关系：关系型数据库
- 映射： 二者之间的对应关系

| Java概念 | 数据库概念 |
| :------: | ---------- |
|    类    | 表         |
|   属性   | 字段/列    |
|   对象   | 记录/行    |

1. 映射文件的命名规则

   表所对应的实体类的类名+Mapper.xml 例如:表t_user，映射的实体类为User，所对应的映射文件为UserMapper.xml 因此一个映射文件对应一个实体类，对应一张表的操作 MyBatis映射文件用于编写SQL，访问以及操作表中的数据 MyBatis映射文件存放的位置是src/main/resources/mappers目录下

2. MyBatis中可以面向接口操作数据，要保证两个一致：

   - mapper接口的全类名和映射文件的命名空间(namespace)保持一致
   - mapper接口中方法的方法名和映射文件中编写SQL的标签的id属性保持一致

```xml
<?xml version="1.0" encoding="UTF-8" ?> <!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="top.withlevi.mybatis.mapper.UserMapper">
    <insert id="insertUser">
        insert into u_user(username,password,age,gender,email)
        values ("keria","keria123321","18","1","kerial@t1.com")
    </insert>

    <delete id="deleteUser">
        delete from u_user where id=5;
    </delete>

    <update id="updateUser">
        update u_user set username="Faker" where id=4
    </update>

    <select id="getUserById" resultType="top.withlevi.mybatis.pojo.User">
        select * from u_user where id=1;
    </select>
    
    <select id="getUserList" resultType="top.withlevi.mybatis.pojo.User">
        select  * from  u_user;
    </select>

    <select id="getUserById" resultType="top.withlevi.mybatis.pojo.User">
        select * from u_user where id = #{id}
    </select>
</mapper>
```



#### 6、通过junit测试功能

```java
package top.withlevi.mybatis.test;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;
import top.withlevi.mybatis.mapper.UserMapper;
import top.withlevi.mybatis.pojo.User;

import java.io.IOException;
import java.io.InputStream;
import java.util.List;

/**
 * @author Levi.Zhao
 * @version 1.0
 * @date 2022/4/18 9:06 PM
 */
public class MapperTest {

    @Test
    public void testInsert() throws IOException {
      // 读取MyBatis的核心配置文件
        InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
      // 创建SqlSessionFactoryBuilder对象
        SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
      //通过核心配置文件所对应的字节输入流创建工厂类SqlSessionFactory，生产SqlSession对象
        SqlSessionFactory build = sqlSessionFactoryBuilder.build(is);
      //创建SqlSession对象，此时通过SqlSession对象所操作的sql都必须手动提交或回滚事务 
      //SqlSession sqlSession = sqlSessionFactory.openSession(); 
      //创建SqlSession对象，此时通过SqlSession对象所操作的sql都会自动提交
        SqlSession sqlSession = build.openSession(true);
      //通过代理模式创建UserMapper接口的代理实现类对象
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        int i = mapper.insertUser();
        if (i>0){
       //调用UserMapper接口中的方法，就可以根据UserMapper的全类名匹配元素文件，通过调用的方法名匹配 映射文件中的SQL标						签，并执行标签中的SQL语句
            System.out.println("Insert user successful.");
        }
    }



    @Test
    public void testDelete() throws IOException {
        InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
        SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
        SqlSessionFactory build = sqlSessionFactoryBuilder.build(is);
        SqlSession sqlSession = build.openSession(true);
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        int i = mapper.deleteUser();
        if (i>0){
            System.out.println("Delete user successful.");
        }
    }


    @Test
    public void testUpdate() throws IOException {
        InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
        SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
        SqlSessionFactory build = sqlSessionFactoryBuilder.build(is);
        SqlSession sqlSession = build.openSession(true);
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        int i = mapper.updateUser();
        if (i>0){
            System.out.println("Update user successful.");
        }
    }


    @Test
    public void testGetUserById() throws IOException {
        InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
        SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
        SqlSessionFactory build = sqlSessionFactoryBuilder.build(is);
        SqlSession sqlSession = build.openSession(true);
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        User user = mapper.getUserById();
        System.out.println(user);

    }


    @Test
    public void testgetUserList() throws IOException {
        InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
        SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
        SqlSessionFactory build = sqlSessionFactoryBuilder.build(is);
        SqlSession sqlSession = build.openSession(true);
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        List<User> userList = mapper.getUserList();
        for (User user : userList) {
            System.out.println(user);
        }

    }

    @Test
    public void testGetUserById2() throws IOException {
        InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
        SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
        SqlSessionFactory build = sqlSessionFactoryBuilder.build(is);
        SqlSession sqlSession = build.openSession(true);
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        User user = mapper.getUserById(3);
        System.out.println(user);

    }
}

```

>- SqlSession:代表Java程序和**数据库**之间的**会话**。(HttpSession是Java程序和浏览器之间的会话
>- SqlSessionFactory:是“生产”SqlSession的“工厂”。
>- 工厂模式:如果创建某一个对象，使用的过程基本固定，那么我们就可以把创建这个对象的 相关代码封装到一个“工厂类”中，以后都使用这个工厂类来“生产”我们需要的对象。



#### 7、加入log4j日志功能

1. 加入依赖

   ```xml
    <!-- log4j日志 -->
           <dependency>
               <groupId>log4j</groupId>
               <artifactId>log4j</artifactId>
               <version>1.2.17</version>
           </dependency>
   ```

2. 加入log4j的配置文件

   > log4j的配置文件名为log4j.xml，存放的位置是src/main/resources目录下

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">
<log4j:configuration xmlns:log4j="http://jakarta.apache.org/log4j/">
    <appender name="STDOUT" class="org.apache.log4j.ConsoleAppender"> <param name="Encoding" value="UTF-8" />
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%-5p %d{MM-dd HH:mm:ss,SSS} %m (%F:%L) \n" />
        </layout>
    </appender>
    <logger name="java.sql"> <level value="debug" />
    </logger>
    <logger name="org.apache.ibatis">
        <level value="info" />
    </logger>
    <root>
        <level value="debug" /> <appender-ref ref="STDOUT" />
    </root>
</log4j:configuration>
```

**日志的级别*

> FATAL(致命)>ERROR(错误)>WARN(警告)>INFO(信息)>DEBUG(调试) 从左到右打印的内容越来越详细



### 三、核心配置文件详解

核心配置文件中的标签必须按照固定的顺序:

> properties?,settings?,typeAliases?,typeHandlers?,objectFactory?,objectWrapperFactory?,reflectorF actory?,plugins?,environments?,databaseIdProvider?,mappers?

```xml
<?xml version="1.0" encoding="UTF-8" ?> <!DOCTYPE configuration
PUBLIC "-//MyBatis.org//DTD Config 3.0//EN" "http://MyBatis.org/dtd/MyBatis-3-config.dtd">

<configuration>
<!--引入properties文件，此时就可以${属性名}的方式访问属性值--> 
  <properties resource="jdbc.properties"></properties>
  
<settings>
<!--将表中字段的下划线自动转换为驼峰-->
<setting name="mapUnderscoreToCamelCase" value="true"/> <!--开启延迟加载-->
<setting name="lazyLoadingEnabled" value="true"/>
</settings>
    <typeAliases>
      
<!--
typeAlias:设置某个具体的类型的别名
属性:
type:需要设置别名的类型的全类名 alias:设置此类型的别名，若不设置此属性，该类型拥有默认的别名，即类名且不区分大小
若设置此属性，此时该类型的别名只能使用alias所设置的值 -->
<!--<typeAlias type="com.atguigu.mybatis.bean.User"></typeAlias>-->
<!--<typeAlias type="com.atguigu.mybatis.bean.User" alias="abc"> </typeAlias>-->
<!--以包为单位，设置改包下所有的类型都拥有默认的别名，即类名且不区分大小写-->
      
<package name="com.atguigu.mybatis.bean"/> </typeAliases>
  
<!-- environments:设置多个连接数据库的环境 属性:
default:设置默认使用的环境的id
-->
  
<environments default="mysql_test"> 

<!--environment:设置具体的连接数据库的环境信息
属性: id:设置环境的唯一标识，可通过environments标签中的default设置某一个环境的id，
表示默认使用的环境
-->
  
<environment id="mysql_test"> 
  
<!--transactionManager:设置事务管理方式 属性:
type:设置事务管理方式，type="JDBC|MANAGED" type="JDBC":设置当前环境的事务管理都必须手动处理 type="MANAGED":设置事务被管理，例如spring中的AOP-->
  
<transactionManager type="JDBC"/> 
  
<!--
dataSource:设置数据源
属性:
type:设置数据源的类型，type="POOLED|UNPOOLED|JNDI" type="POOLED":使用数据库连接池，即会将创建的连接进行缓存，下次使用可以从缓存中直接获取，不需要重新创建 type="UNPOOLED":不使用数据库连接池，即每次使用连接都需要重新创建
type="JNDI":调用上下文中的数据源 -->
  
<dataSource type="POOLED">
<!--设置驱动类的全类名-->
<property name="driver" value="${jdbc.driver}"/> <!--设置连接数据库的连接地址-->
<property name="url" value="${jdbc.url}"/> <!--设置连接数据库的用户名-->
<property name="username" value="${jdbc.username}"/> <!--设置连接数据库的密码-->
<property name="password" value="${jdbc.password}"/>
 </dataSource>
  </environment>
  </environments>
  
<!--引入映射文件--> 
<mappers>
<mapper resource="UserMapper.xml"/> <!--
       以包为单位，将包下所有的映射文件引入核心配置文件
注意:此方式必须保证mapper接口和mapper映射文件必须在相同的包下 -->
<package name="com.atguigu.mybatis.mapper"/> 
</mappers>
</configuration>
```



### 四、MyBatis的增删改查

1. 添加

   ```xml
    <insert id="insertUser">
           insert into u_user(username,password,age,gender,email)
           values ("keria","keria123321","18","1","kerial@t1.com")
       </insert>
   
   ```

2. 删除

   ```xml
     <delete id="deleteUser">
           delete from u_user where id=5;
       </delete>
   ```

3. 修改

   ```xml
    <update id="updateUser">
           update u_user set username="Faker" where id=4
       </update>
   ```

4. 查看一个实体类对象

   ```xml
    <select id="getUserById" resultType="top.withlevi.mybatis.pojo.User">
           select * from u_user where id=1;
       </select>
       
   ```

5. 查询集合

   ```xml
     <select id="getUserList" resultType="top.withlevi.mybatis.pojo.User">
           select  * from  u_user;
       </select>
   ```

> 注意:
>
> 1、查询的标签select必须设置属性resultType或resultMap，用于设置实体类和数据库表的映射 关系
>
> resultType:自动映射，用于属性名和表中字段名一致的情况 resultMap:自定义映射，用于一对多或多对一或字段名和属性名不一致的情况
>
> 2、当查询的数据为多条时，不能使用实体类作为返回值，只能使用集合，否则会抛出异常 TooManyResultsException;但是若查询的数据只有一条，可以使用实体类或集合作为返回值



### 五、**MyBatis获取参数值的两种方式(重点)**

MyBatis获取参数值的两种方式:**${}**和**#{}** 

${}的本质就是字符串拼接，#{}的本质就是占位符赋值

${}使用字符串拼接的方式拼接sql，若为字符串类型或日期类型的字段进行赋值时，需要手动加单引号;

但是#{}使用占位符赋值的方式拼接sql，此时为字符串类型或日期类型的字段进行赋值时，可以自动添加单引号

#### 1、单个字面量类型的参数

若mapper接口中的方法参数为单个的字面量类型

此时可以使用${}和#{}以任意的名称获取参数的值，注意${}需要手动加单引号



#### 2、多个字面量类型的参数

若mapper接口中的方法参数为多个时

此时MyBatis会自动将这些参数放在一个map集合中，以arg0,arg1...为键，以参数为值;以 param1,param2...为键，以参数为值;因此只需要通过${}和#{}访问map集合的键就可以获取相对应的 值，注意${}需要手动加单引号



#### 3、map集合类型的参数

若mapper接口中的方法需要的参数为多个时，此时可以手动创建map集合，将这些数据放在map中

只需要通过${}和#{}访问map集合的键就可以获取相对应的值，注意${}需要手动加单引号



#### 4、实体类类型的参数

若mapper接口中的方法参数为实体类对象时 此时可以使用${}和#{}，通过访问实体类对象中的属性名获取属性值，注意${}需要手动加单引号



#### 5、使用@Param标识参数

可以通过@Param注解标识mapper接口中的方法参数

此时，会将这些参数放在map集合中，以@Param注解的value属性值为键，以参数为值;以 param1,param2...为键，以参数为值;只需要通过${}和#{}访问map集合的键就可以获取相对应的值， 注意${}需要手动加单引号



### 六、Mybatis的各种查询功能

#### 1、查询一个实体类

```java
    /**
     *  根据用户id查询用户信息
     */
    User getUserInfoById(@Param("id") int id);
```

```xml
    <!--Get User Info by ID-->

    <select id="getUserInfoById" resultType="user">
        select * from u_user where id=#{id}
    </select>
```

#### 2、查询一个list集合

```java
 		/**
     * 查询所有的用户信息
     */

    List<User> listAllUserInfo();
```

```xml

    <!--select all list user information-->

    <select id="listAllUserInfo" resultType="user">
        select * from u_user;
    </select>
```

#### 3、查询单个数据

```java
  	/**
     * 查询单个数据 统计用户数量
     */
    int getCount();
```

```xml
 <!--get User Count-->
    <select id="getCount" resultType="int">
        select count(*) from u_user;
    </select>
```

#### 4、查询一条数据为map集合

```java
 		/**
     * 查询数据为Map集合
     */

    Map<String,Object> getUserToMap(@Param("id") int id);
```

```xml
 <!--get map collection data by id-->
    <select id="getUserToMap" resultType="map">
        select * from u_user where id=#{id}
    </select>
```

#### 5、查询多条数据为map集合

**方式一**

```java

    /**
     *  查询多条数据为map集合
     */
    List<Map<String,Object>> getALlUserToMap();

```

```xml

    <!--get ALl User Info to Map-->
    <select id="getALlUserToMap" resultType="map">
        select  * from  u_user;
    </select>
```

**方式二*

```java
 		/**
     *  查询多条数据为map集合 设置键值对
     */
   @MapKey("id")
   Map<String,Object> getALlToMap();
```

```xml

    <!--get all user info to map(set key-value)-->
    <select id="getALlToMap" resultType="map">
        select  * from  u_user;
    </select>
```



### 七、特殊SQL的执行

#### 1、模糊查询

```java
		/**
     * 模糊查询
     */
    List<User> fuzzyQuery(@Param("fuzzy") String fuzzy);
```

```xml
<!--Fuzzy query-->
    <select id="fuzzyQuery" resultType="user">
        <!--select * from u_user where username like "%${fuzzy}%"-->
        <!--select * from u_user where username like concat('%',#{fuzzy},'%')-->
        select * from u_user where username like "%"#{fuzzy}"%"
    </select>
```

#### 2、批量删除

```java
		/**
     *批量删除
     */
    int deleteMore(@Param("ids") String ids);
```

```xml
 <!--delete more-->
    <delete id="deleteMore">
        delete  from u_user where id in (${ids})
    </delete>
```

#### 3、动态设置表名

```java
		/**
     * 动态设置表名
     */
    List<User> getAllUser(@Param("tablename") String tablename);
```

```xml
 <!--get all user-->
    <select id="getAllUser" resultType="user">
        select * from ${tablename}
    </select>
```

#### 4、添加功能获取自增的主键

t_clazz(clazz_id,clazz_name) 

t_student(student_id,student_name,clazz_id) 

1、添加班级信息

2、获取新添加的班级的id 

3、为班级分配学生，即将某学的班级id修改为新添加的班级的id

```java
/**
* 添加用户信息
* @param user
* @return
* useGeneratedKeys:设置使用自增的主键
* keyProperty:因为增删改有统一的返回值是受影响的行数，因此只能将获取的自增的主键放在传输的参
数user对象的某个属性中 */
int insertUser(User user);

```

```xml
<!--int insertUser(User user);-->
<insert id="insertUser" useGeneratedKeys="true" keyProperty="id">
    insert into t_user values(null,#{username},#{password},#{age},#{sex})
</insert>
```



补充工具类-Utils.java

```java
package top.withlevi.mybatis.utils;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;

public class SqlSessionUtils {

    public static SqlSession getSqlSession() {
        SqlSession sqlSession = null;

        try {
            InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
            SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
            SqlSessionFactory build = sqlSessionFactoryBuilder.build(is);
            sqlSession = build.openSession(true);
        } catch (IOException e) {
            e.printStackTrace();
        }

        return sqlSession;
    }

}

```



### 八、自定义映射resultMap

#### 1、resultMap处理字段和属性的映射关系

若字段名和实体类中的属性名不一致，则可以通过resultMap设置自定义映射

EmpMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?> <!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="top.withlevi.mybatis.mapper.EmpMapper">

    <select id="getALlEmp1" resultType="emp">
        select *
        from t_emp;
    </select>

    <resultMap id="userMap" type="Emp">
        <id property="eid" column="eid"/>
        <result property="empName" column="emp_name"/>
        <result property="age" column="age"/>
        <result property="gender" column="gender"/>
        <result property="email" column="email"/>
    </resultMap>

    <!--Get All Emp-->
    <select id="getALlEmp" resultMap="userMap">
        select *
        from t_emp;
    </select>

    <!-- Emp getEmpInfoAndDeptById(@Param("eid") String eid)-->
    <!--多对一映射处理方式一：级联方式处理映射关系-->
    <resultMap id="EmpAndDept" type="emp">
        <id property="eid" column="eid"/>
        <result property="empName" column="emp_name"/>
        <result property="age" column="age"/>
        <result property="gender" column="gender"/>
        <result property="email" column="email"/>
        <result property="dept.deptId" column="dept_id"/>
        <result property="dept.deptName" column="dept_name"/>
    </resultMap>

    <select id="getEmpInfoAndDeptById1" resultMap="EmpAndDept">
        select *
        from t_emp e
                 left join t_dept d on e.eid = d.dept_id
        where e.eid = #{eid}
    </select>

    <!--多对一处理映射方式二: association-->

    <resultMap id="EmpAndDeptAssociation" type="emp">
        <id property="eid" column="eid"/>
        <result property="empName" column="emp_name"/>
        <result property="age" column="age"/>
        <result property="gender" column="gender"/>
        <result property="email" column="email"/>
        <association property="dept" javaType="Dept">
            <id property="deptId" column="dept_id"/>
            <result property="deptName" column="dept_name"/>
        </association>
    </resultMap>

    <select id="getEmpInfoAndDeptById" resultMap="EmpAndDeptAssociation">
        select *
        from t_emp e
                 left join t_dept d on e.eid = d.dept_id
        where e.eid = #{eid};
    </select>


    <!--多对一处理映射方式三：分步查询-->
    <resultMap id="empDeptStepMap" type="emp">
        <id property="eid" column="eid"/>
        <result property="empName" column="emp_name"/>
        <result property="age" column="age"/>
        <result property="gender" column="gender"/>
        <result property="email" column="email"/>
        <association property="dept" select="top.withlevi.mybatis.mapper.DeptMapper.getEmpDeptByStep" column="dept_id">

        </association>
    </resultMap>


    <select id="getEmpByStep" resultMap="empDeptStepMap">
        select *
        from t_emp
        where eid = #{eid}
    </select>

    <!-- List<Emp> getEmpListByDeptId(@Param("deptId") String deptId)-->
    <select id="getEmpListByDeptId" resultType="emp">
        select * from t_emp where dept_id=#{deptId}
    </select>
</mapper>
```

EmpMapper.java

```java
package top.withlevi.mybatis.mapper;

import org.apache.ibatis.annotations.Param;
import top.withlevi.mybatis.pojo.Emp;

import java.util.List;

public interface EmpMapper {

    /**
     *获取所有的emp
     *
     */
    List<Emp> getALlEmp();

    /**
     *  根据id查询emp的信息以及对应的部门
     */

    Emp getEmpInfoAndDeptById(@Param("eid") Integer eid);


    /**
     * 通过分步查询查询员工信息
     */

    Emp getEmpByStep(@Param("eid") Integer eid);


    /**
     * 根据部门id查询部门中所有员工
     */

    List<Emp> getEmpListByDeptId(@Param("deptId") String deptId);
}

```

> 若字段名和实体类中的属性名不一致，但是字段名符合数据库的规则(使用_)，实体类中的属性 名符合Java的规则(使用驼峰)
>
> 此时也可通过以下两种方式处理字段名和实体类中的属性的映射关系 a>可以通过为字段起别名的方式，保证和实体类中的属性名保持一致
>
> b>可以在MyBatis的核心配置文件中设置一个全局配置信息mapUnderscoreToCamelCase，可 以在查询表中数据时，自动将_类型的字段名转换为驼峰
>
> 例如:字段名user_name，设置了mapUnderscoreToCamelCase，此时字段名就会转换为 userName

#### 2、多对一映射处理

> 查询员工信息以及员工对应的部门信息

1. 级联方式处理映射关系

   ```java
   		/**
        *  根据id查询emp的信息以及对应的部门
        */
   
       Emp getEmpInfoAndDeptById(@Param("eid") Integer eid);
   ```

   

   ```xml
    <!-- Emp getEmpInfoAndDeptById(@Param("eid") String eid)-->
       <!--多对一映射处理方式一：级联方式处理映射关系-->
       <resultMap id="EmpAndDept" type="emp">
           <id property="eid" column="eid"/>
           <result property="empName" column="emp_name"/>
           <result property="age" column="age"/>
           <result property="gender" column="gender"/>
           <result property="email" column="email"/>
           <result property="dept.deptId" column="dept_id"/>
           <result property="dept.deptName" column="dept_name"/>
       </resultMap>
   
       <select id="getEmpInfoAndDeptById1" resultMap="EmpAndDept">
           select *
           from t_emp e
                    left join t_dept d on e.eid = d.dept_id
           where e.eid = #{eid}
       </select>
   
   ```

2. 使用association处理映射关系

   ```java
   		/**
        *  根据id查询emp的信息以及对应的部门
        */
   
       Emp getEmpInfoAndDeptById(@Param("eid") Integer eid);
   ```

   

   ```xml
   <!--多对一处理映射方式二: association-->
   
       <resultMap id="EmpAndDeptAssociation" type="emp">
           <id property="eid" column="eid"/>
           <result property="empName" column="emp_name"/>
           <result property="age" column="age"/>
           <result property="gender" column="gender"/>
           <result property="email" column="email"/>
           <association property="dept" javaType="Dept">
               <id property="deptId" column="dept_id"/>
               <result property="deptName" column="dept_name"/>
           </association>
       </resultMap>
   
       <select id="getEmpInfoAndDeptById" resultMap="EmpAndDeptAssociation">
           select *
           from t_emp e
                    left join t_dept d on e.eid = d.dept_id
           where e.eid = #{eid};
       </select>
   
   ```

3. 分步查询

   - 查询员工信息

     ```java
     /**
          * 通过分步查询查询员工信息
          */
     
         Emp getEmpByStep(@Param("eid") Integer eid);
     ```

     

     ```xml
      <!--多对一处理映射方式三：分步查询-->
         <resultMap id="empDeptStepMap" type="emp">
             <id property="eid" column="eid"/>
             <result property="empName" column="emp_name"/>
             <result property="age" column="age"/>
             <result property="gender" column="gender"/>
             <result property="email" column="email"/>
             <association property="dept" select="top.withlevi.mybatis.mapper.DeptMapper.getEmpDeptByStep" column="dept_id">
     
             </association>
         </resultMap>
     
     
         <select id="getEmpByStep" resultMap="empDeptStepMap">
             select *
             from t_emp
             where eid = #{eid}
         </select>
     ```

   - 根据员工所对应的部门id查询部门信息

     ```java
     		/**
          * 分步查询第二步：根据员工id查询对应的部门信息
          *
          */
     
         Dept getEmpDeptByStep(@Param("deptId") Integer deptID);
     ```

     ```xml
      <!--Get emp and dept by step-->
         <select id="getEmpDeptByStep" resultType="dept">
             select  * from t_dept where dept_id=#{deptId}
         </select>
     ```



#### 3、一对多映射处理

1. collection

   ```java
    		/**
        * 根据部门id查询部门以及部门中的员工信息
        */
   
       Dept getDeptEmpByDeptId(@Param("deptId") Integer deptId);
   ```

   ```xml
     <!-- Dept getDeptEmpByDeptId(@Param("deptId") String deptId)-->
   
       <resultMap id="deptEmpMap" type="dept">
           <id property="deptId" column="dept_id"/>
           <result property="deptName" column="dept_name"/>
   
           <collection property="emps" ofType="emp">
               <id property="eid" column="eid"/>
               <result property="empName" column="emp_name"/>
               <result property="age" column="age"/>
               <result property="gender" column="gender"/>
               <result property="email" column="email"/>
               <!--<result property="dept.deptId" column="dept_id"/>
               <result property="dept.deptName" column="dept_name"/>-->
              <!-- <association property="dept" javaType="Dept">
                   <id property="deptId" column="dept_id"/>
                   <result property="deptName" column="dept_name"/>
               </association>-->
           </collection>
       </resultMap>
   
       <select id="getDeptEmpByDeptId" resultMap="deptEmpMap">
           select * from t_dept d left join t_emp e on d.dept_id = e.dept_id where d.dept_id=#{deptId}
       </select>
   ```

2. 分步查询

   - 查询部门信息

     ```java
     		/**
          * 分步查询部门和部门中的员工
          */
     
         Dept getDeptByStep(@Param("deptId") Integer deptId);
     
     ```

     ```xml
      <resultMap id="deptEmpStep" type="dept">
             <id property="deptId" column="dept_id"/>
             <result property="deptName" column="dept_name"/>
             <collection property="emps" fetchType="eager" select="top.withlevi.mybatis.mapper.EmpMapper.getEmpListByDeptId" column="dept_id"/>
         </resultMap>
     
     
         <!-- Dept getDeptByStep(@Param("deptId") String deptId)-->
         <select id="getDeptByStep" resultMap="deptEmpStep">
             select * from t_dept where dept_id=#{deptId}
         </select>
     ```

   - 根据部门id查询部门中的所有员工

     ```java
     /**
          * 根据部门id查询部门中所有员工
          */
     
         List<Emp> getEmpListByDeptId(@Param("deptId") String deptId);
     ```

     ```xml
     <!-- List<Emp> getEmpListByDeptId(@Param("deptId") String deptId)-->
         <select id="getEmpListByDeptId" resultType="emp">
             select * from t_emp where dept_id=#{deptId}
         </select>
     ```

> 分步查询的优点:可以实现延迟加载，但是必须在核心配置文件中设置全局配置信息: lazyLoadingEnabled:延迟加载的全局开关。当开启时，所有关联对象都会延迟加载
>
> aggressiveLazyLoading:当开启时，任何方法的调用都会加载该对象的所有属性,否则，每个 属性会按需加载
>
> 此时就可以实现按需加载，获取的数据是什么，就只会执行相应的sql。此时可通过association和 collection中的fetchType属性设置当前的分步查询是否使用延迟加载，fetchType="lazy(延迟加 载)|eager(立即加载)"

Mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?> <!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <!--引入外部数据库文件-->
    <properties resource="jdbc.properties"/>


    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <setting name="lazyLoadingEnabled" value="true"/>
    </settings>



    <typeAliases>
        <package name="top.withlevi.mybatis.pojo"/>
    </typeAliases>
    
    <!--设置连接数据库的环境-->
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url"
                          value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>
    <!--引入映射文件-->
    <mappers>
        <!-- <mapper resource="UserMappers/UserMapper.xml"/>-->
        <package name="top.withlevi.mybatis.mapper"/>
    </mappers>

</configuration>
```



### 九、动态SQL

Mybatis框架的动态SQL技术是一种根据特定条件动态拼装SQL语句的功能，它存在的意义是为了解决

拼接SQL语句字符串时的痛点问题。

#### 1、if

if标签可通过test属性的表达式进行判断，若表达式的结果为true，则标签中的内容会执行;反之标签中 的内容不会执行

```java
 		/**
     * 多条件查询
     */

    List<Emp> getEmpByCondition(Emp emp);
```

```xml
 <!--List<Emp> getEmpByCondition(Emp emp)-->
    <select id="getEmpByConditionOne" resultType="emp">
        select * from t_emp where 1=1
        <if test="empName !='' and empName !=null">
            and emp_name =#{empName}
        </if>
        <if test="age!='' and age !=null">
            and age=#{age}
        </if>
        <if test="gender !='' and gender !=null">
            and gender =#{gender }
        </if>
        <if test="email !='' and email !=null">
            and email =#{email}
        </if>
    </select>
```

#### 2、where

```java
		 /**
     * 多条件查询
     */

    List<Emp> getEmpByCondition(Emp emp);
```

```xml
<select id="getEmpByConditionTwo" resultType="emp">
        select * from t_emp
        <where>
            <if test="empName !='' and empName !=null">
                and emp_name =#{empName}
            </if>
            <if test="age!='' and age !=null">
                and age=#{age}
            </if>
            <if test="gender !='' and gender !=null">
                or gender =#{gender }
            </if>
            <if test="email !='' and email !=null">
                and email =#{email}
            </if>
        </where>
    </select>
```

> where和if一般结合使用: a>若where标签中的if条件都不满足，则where标签没有任何功能，即不会添加where关键字
>
> b>若where标签中的if条件满足，则where标签会自动添加where关键字，并将条件最前方多余的 and去掉
>
> 注意:where标签不能去掉条件最后多余的and

#### 3、trim

```java
		/**
     * 多条件查询
     */

    List<Emp> getEmpByCondition(Emp emp);
```

```xml
 <select id="getEmpByCondition" resultType="emp">
        select * from t_emp
        <trim prefix="where" suffixOverrides="and|or">
            <if test="empName !='' and empName !=null">
                emp_name =#{empName} and
            </if>
            <if test="age!='' and age !=null">
                age=#{age} and
            </if>
            <if test="gender !='' and gender !=null">
                gender =#{gender } or
            </if>
            <if test="email !='' and email !=null">
                email =#{email}
            </if>
        </trim>
    </select>
```

> trim用于去掉或添加标签中的内容
> 常用属性: prefix:在trim标签中的内容的前面添加某些内容
>
>  prefixOverrides:在trim标签中的内容的前面去掉某些内容 
>
> suffix:在trim标签中的内容的后面添加某些内容 
>
> suffixOverrides:在trim标签中的内容的后面去掉某些内容

#### 4、**choose****、****when****、****otherwise**

choose、when、otherwise相当于if...else if..else

```java
		 /**
     * 测试choose、when、otherwise
     */

    List<Emp> getEmpByChoose(Emp emp);
```

```xml
 <!--List<Emp> getEmpByChoose(Emp emp)-->
    <select id="getEmpByChoose" resultType="emp">
        select * from t_emp
        <where>
            <choose>
                <when test="empName != '' and empName !=null">
                    emp_name = #{empName}
                </when>
                <when test="age != '' and age != null">
                    age = #{age}
                </when>
                <when test="gender != '' and gender != null">
                    gender = #{gender}
                </when>
                <when test="email != '' and email!= null">
                    email= #{email}
                </when>
                <otherwise>
                    dept_id=1
                </otherwise>
            </choose>
        </where>
    </select>
```

#### 5、foreach

```java
    /**
     * 通过数组实现批量删除-foreach
     */

    int batchDel(@Param("eids") Integer[] eids);
```

```xml
  <!--int batchDel(@Param("eids") Integer[] eids)-->
    <delete id="batchDel">
        <!--delete from t_emp where eid in
            <foreach collection="eids" item="eid" separator="," open="(" close=")">
                 #{eid}
             </foreach>-->
        delete from t_emp where
        <foreach collection="eids" item="eid" separator="or">
            eid = #{eid}
        </foreach>
    </delete>
```



```java
 /**
     * 通过list集合实现批量添加
     */
    int batchAdd(@Param("emps") List<Emp> emps);
```

```xml
 <!--  int batchAdd(List<Emp> emps)-->
    <insert id="batchAdd">
        insert into t_emp values
        <foreach collection="emps" item="emp" separator=",">
            (null,#{emp.empName},#{emp.age},#{emp.gender},#{emp.email},null)
        </foreach>
    </insert>
```



#### 6、SQL片段

sql片段，可以记录一段公共sql片段，在使用的地方通过include标签进行引入

```java
 /**
     * 查询所有信息
     */

   List<Emp> queryAllInfo();
```

```xml
<sql id="empColumns">eid,emp_name,age,gender,email</sql>

    <!-- int queryAllInfo()-->
    <select id="queryAllInfo" resultType="emp">
        select <include refid="empColumns"></include> from t_emp
    </select>
```





### 十、MyBatis的缓存

#### 1、MyBatis的一级缓存

一级缓存是SqlSession级别的，通过同一个SqlSession查询的数据会被缓存，下次查询相同的数据，就 会从缓存中直接获取，不会从数据库重新访问

使一级缓存失效的四种情况:
 \1) 不同的SqlSession对应不同的一级缓存
 \2) 同一个SqlSession但是查询条件不同
 \3) 同一个SqlSession两次查询期间执行了任何一次增删改操作 4) 同一个SqlSession两次查询期间手动清空了缓存



#### 2、MyBatis的二级缓存

二级缓存是SqlSessionFactory级别，通过同一个SqlSessionFactory创建的SqlSession查询的结果会被

缓存;此后若再次执行相同的查询语句，结果就会从缓存中获取
 二级缓存开启的条件: 

a>在核心配置文件中，设置全局配置属性cacheEnabled="true"，默认为true，不需要设置

 b>在映射文件中设置标签<cache />

c>二级缓存必须在SqlSession关闭或提交之后有效 

d>查询的数据所转换的实体类类型必须实现序列化的接口 使二级缓存失效的情况: 两次查询之间执行了任意的增删改，会使一级和二级缓存同时失效



#### 3、二级缓存的相关配置

在mapper配置文件中添加的cache标签可以设置一些属性:

- eviction属性:缓存回收策略

  LRU(Least Recently Used) – 最近最少使用的:移除最长时间不被使用的对象。

  FIFO(First in First out) – 先进先出:按对象进入缓存的顺序来移除它们。

  SOFT – 软引用:移除基于垃圾回收器状态和软引用规则的对象。

  WEAK – 弱引用:更积极地移除基于垃圾收集器状态和弱引用规则的对象。

  默认的是 LRU。

- flushInterval属性:刷新间隔，单位毫秒

  默认情况是不设置，也就是没有刷新间隔，缓存仅仅调用语句时刷新

- size属性:引用数目，正整数

  代表缓存最多可以存储多少个对象，太大容易导致内存溢出

- readOnly属性:只读，true/false

  true:只读缓存;会给所有调用者返回缓存对象的相同实例。因此这些对象不能被修改。这提供了 很重要的性能优势。

  false:读写缓存;会返回缓存对象的拷贝(通过序列化)。这会慢一些，但是安全，因此默认是 false。



#### 4、MyBatis缓存查询的顺序

- 先查询二级缓存，因为二级缓存中可能会有其他程序已经查出来的数据，可以拿来直接使用。

-  如果二级缓存没有命中，再查询一级缓存

- 如果一级缓存也没有命中，则查询数据库

- SqlSession关闭之后，一级缓存中的数据会写入二级缓存

  

#### 5、整合第三方缓存EHCache

- 添加依赖

  ```xml
  
          <!-- Mybatis EHCache整合包 -->
          <dependency>
              <groupId>org.mybatis.caches</groupId>
              <artifactId>mybatis-ehcache</artifactId>
              <version>1.2.1</version>
          </dependency>
          <!-- slf4j日志门面的一个具体实现 -->
          <dependency>
              <groupId>ch.qos.logback</groupId>
              <artifactId>logback-classic</artifactId>
              <version>1.2.3</version>
          </dependency>
  ```

- 各jar包功能

  | jar包名称       | 作用                            |
  | --------------- | ------------------------------- |
  | mybatis-ehcache | Mybatis和EHCache的整合包        |
  | ehcache         | EHCache核心包                   |
  | slf4j-api       | SLF4J日志门面包                 |
  | logback-classic | 支持SLF4J门面接口的一个具体实现 |

- 创建EHCache的配置文件ehcache.xml

  ```xml
  <?xml version="1.0" encoding="utf-8" ?>
  <ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="../config/ehcache.xsd">
      <!-- 磁盘保存路径 -->
      <diskStore path="/Users/levi.zhao/Desktop/Project/Java_Project/mybatis-study/mybatis-demo03/src/main/java/top/withlevi/mybatis/CacheData"/>
      <defaultCache
              maxElementsInMemory="1000"
              maxElementsOnDisk="10000000"
              eternal="false"
              overflowToDisk="true"
              timeToIdleSeconds="120"
              timeToLiveSeconds="120"
              diskExpiryThreadIntervalSeconds="120"
              memoryStoreEvictionPolicy="LRU">
      </defaultCache>
  </ehcache>
  ```

- 设置二级缓存的类型

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?> <!DOCTYPE mapper
          PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  <mapper namespace="top.withlevi.mybatis.mapper.CacheMapper">
  
      <cache eviction="FIFO" type="org.mybatis.caches.ehcache.EhcacheCache"/>
  
      <!--Emp getEmpById(@Param("eid") String eid)-->
      <select id="getEmpById" resultType="Emp">
          select  * from  t_emp where eid=#{eid}
      </select>
  
  
      <!--void insertEmp(Emp emp)-->
      <insert id="insertEmp">
          insert into t_emp values (null,#{empName},#{age},#{gender},#{email},null)
      </insert>
  
  
  </mapper>
  ```

- 加入logback日志

  存在SLF4J时，作为简易日志的log4j将失效，此时我们需要借助SLF4J的具体实现logback来打印日志。

  创建logback的配置文件logback.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <configuration debug="true">
      <!-- 指定日志输出的位置 -->
      <appender name="STDOUT"
      class="ch.qos.logback.core.ConsoleAppender">
          <encoder>
          <!-- 日志输出的格式 -->
          <!-- 按照顺序分别是:时间、日志级别、线程名称、打印日志的类、日志主体内容、换行 -->
          <pattern>[%d{HH:mm:ss.SSS}] [%-5level] [%thread] [%logger] [%msg]%n</pattern>
              <charset>UTF-8</charset>
          </encoder>
      </appender>
      <!-- 设置全局日志级别。日志级别按顺序分别是:DEBUG、INFO、WARN、ERROR -->
      <!-- 指定任何一个日志级别都只打印当前级别和后面级别的日志。 -->
      <root level="DEBUG">
          <!-- 指定打印日志的appender，这里通过“STDOUT”引用了前面配置的appender -->
          <appender-ref ref="STDOUT" /> </root>
      <!-- 根据特殊需求指定局部日志级别 -->
      <logger name="top.withlevi.mybatis.mapper" level="DEBUG"/>
  </configuration>
  ```

- **EHCache****配置文件说明**

| 属性名                          | 是否必须 |                             作用                             |
| ------------------------------- | -------- | :----------------------------------------------------------: |
| maxElementsInMemory             | 是       |               在内存中缓存的element的最大数目                |
| maxElementsOnDisk               | 是       |      在磁盘上缓存的element的最大数目，若是0表示无 穷大       |
| eternal                         | 是       | 设定缓存的elements是否永远不过期。 如果为 true，则缓存的数据始终有效， 如果为false那么还 要根据timeToIdleSeconds、timeToLiveSeconds 判断 |
| overflowToDisk                  | 是       |   设定当内存缓存溢出的时候是否将过期的element 缓存到磁盘上   |
| timeToIdleSeconds               | 否       | 当缓存在EhCache中的数据前后两次访问的时间超 过timeToIdleSeconds的属性取值时， 这些数据便 会删除，默认值是0,也就是可闲置时间无穷大 |
| timeToLiveSeconds               | 否       | 缓存element的有效生命期，默认是0.,也就是 element存活时间无穷大 |
| diskSpoolBufferSizeMB           | 否       | DiskStore(磁盘缓存)的缓存区大小。默认是 30MB。每个Cache都应该有自己的一个缓冲区 |
| diskPersistent                  | 否       | 在VM重启的时候是否启用磁盘保存EhCache中的数 据，默认是false。 |
| diskExpiryThreadIntervalSeconds | 否       | 磁盘缓存的清理线程运行间隔，默认是120秒。每 个120s， 相应的线程会进行一次EhCache中数据的 清理工作 |
| memoryStoreEvictionPolicy       | 否       | 当内存缓存达到最大，有新的element加入的时 候， 移除缓存中element的策略。 默认是LRU(最 近最少使用)，可选的有LFU(最不常使用)和 FIFO(先进先出) |





### 十一、MyBatis的逆向工程

- 正向工程:先创建Java实体类，由框架负责根据实体类生成数据库表。Hibernate是支持正向工程的
- 逆向工程:先创建数据库表，由框架负责根据数据库表，反向生成如下资源:
  - Java实体类
  - Mapper接口
  - Mapper映射文件

#### 1、创建逆向工程的步骤

- 添加依赖和插件

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <project xmlns="http://maven.apache.org/POM/4.0.0"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
      <modelVersion>4.0.0</modelVersion>
  
      <groupId>top.withlevi.mybatis</groupId>
      <artifactId>mybatis-demo04</artifactId>
      <version>1.0-SNAPSHOT</version>
      <packaging>jar</packaging>
  
      <properties>
          <maven.compiler.source>8</maven.compiler.source>
          <maven.compiler.target>8</maven.compiler.target>
      </properties>
  
  
      <!-- 依赖MyBatis核心包 -->
      <dependencies>
          <dependency>
              <groupId>org.mybatis</groupId>
              <artifactId>mybatis</artifactId>
              <version>3.5.7</version>
          </dependency>
  
  
          <!-- junit测试 -->
          <dependency>
              <groupId>junit</groupId>
              <artifactId>junit</artifactId>
              <version>4.12</version>
              <scope>test</scope>
          </dependency>
  
          <!-- MySQL驱动 -->
          <dependency>
              <groupId>mysql</groupId>
              <artifactId>mysql-connector-java</artifactId>
              <version>5.1.3</version>
          </dependency>
  
          <!-- slf4j日志门面的一个具体实现 -->
          <dependency>
              <groupId>ch.qos.logback</groupId>
              <artifactId>logback-classic</artifactId>
              <version>1.2.3</version>
          </dependency>
  
          <dependency>
              <groupId>org.jboss</groupId>
              <artifactId>jboss-vfs</artifactId>
              <version>3.2.16.Final</version>
          </dependency>
  
          <!-- https://mvnrepository.com/artifact/com.github.pagehelper/pagehelper -->
          <dependency>
              <groupId>com.github.pagehelper</groupId>
              <artifactId>pagehelper</artifactId>
              <version>5.2.0</version>
          </dependency>
  
      </dependencies>
  
  
      <!-- 控制Maven在构建过程中相关配置 -->
      <build>
          <!-- 构建过程中用到的插件 -->
          <plugins>
              <!-- 具体插件，逆向工程的操作是以构建过程中插件形式出现的 -->
              <plugin>
                  <groupId>org.mybatis.generator</groupId>
                  <artifactId>mybatis-generator-maven-plugin</artifactId>
                  <version>1.3.0</version>
                  <!-- 插件的依赖 -->
                  <dependencies>
                      <!-- 逆向工程的核心依赖 -->
                      <dependency>
                          <groupId>org.mybatis.generator</groupId>
                          <artifactId>mybatis-generator-core</artifactId>
                          <version>1.3.2</version>
                      </dependency>
                      <!-- 数据库连接池 -->
                      <dependency>
                          <groupId>com.mchange</groupId>
                          <artifactId>c3p0</artifactId>
                          <version>0.9.2</version>
                      </dependency>
                      <!-- MySQL驱动 -->
                      <dependency>
                          <groupId>mysql</groupId>
                          <artifactId>mysql-connector-java</artifactId>
                          <version>5.1.8</version>
                      </dependency>
                  </dependencies>
              </plugin>
          </plugins>
      </build>
  
  </project>
  ```

- 创建MyBatis的核心配置文件

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?> <!DOCTYPE configuration
          PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
  <configuration>
  
      <!--引入外部数据库文件-->
      <properties resource="jdbc.properties"/>
  
      <!--设置别名-->
      <typeAliases>
          <package name="top.withlevi.mybatis.pojo"/>
      </typeAliases>
  
      <plugins>
          <!--设置分页插件-->
          <plugin interceptor="com.github.pagehelper.PageInterceptor"/>
      </plugins>
  
  
      <!--设置连接数据库的环境-->
      <environments default="development">
          <environment id="development">
              <transactionManager type="JDBC"/>
              <dataSource type="POOLED">
                  <property name="driver" value="${jdbc.driver}"/>
                  <property name="url"
                            value="${jdbc.url}"/>
                  <property name="username" value="${jdbc.username}"/>
                  <property name="password" value="${jdbc.password}"/>
              </dataSource>
          </environment>
      </environments>
  
      <!--引入映射文件-->
      <mappers>
          <!-- <mapper resource="UserMappers/UserMapper.xml"/>-->
          <package name="top.withlevi.mybatis.mapper"/>
      </mappers>
  </configuration>
  ```

- 创建逆向工程的配置文件

  > 文件名必须是: generatorConfig.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?> <!DOCTYPE generatorConfiguration
          PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
          "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
  <generatorConfiguration>
  
      <!--  targetRuntime: 执行生成的逆向工程的版本
            MyBatis3Simple: 生成基本的CRUD(清新简洁版)
            MyBatis3: 生成带条件的CRUD(奢华尊享版)
       -->
  
  
      <context id="DB2Tables" targetRuntime="MyBatis3">
          <!-- 数据库的连接信息 -->
          <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                          connectionURL="jdbc:mysql://localhost:3306/mybatis"
                          userId="root"
                          password="root123456">
          </jdbcConnection>
  
          <!-- javaBean的生成策略-->
          <javaModelGenerator targetPackage="top.withlevi.mybatis.pojo" targetProject="src/main/java">
              <property name="enableSubPackages" value="true"/>
              <property name="trimStrings" value="true"/>
          </javaModelGenerator>
  
          <!-- SQL映射文件的生成策略 -->
          <sqlMapGenerator targetPackage="top.withlevi.mybatis.mapper"
                           targetProject="src/main/resources">
              <property name="enableSubPackages" value="true"/>
          </sqlMapGenerator>
  
          <!-- Mapper接口的生成策略 -->
          <javaClientGenerator type="XMLMAPPER" targetPackage="top.withlevi.mybatis.mapper"
                               targetProject="src/main/java">
              <property name="enableSubPackages" value="true"/>
          </javaClientGenerator>
  
          <!-- 逆向分析的表 -->
          <!-- tableName设置为*号，可以对应所有表，此时不写domainObjectName -->
          <!-- domainObjectName属性指定生成出来的实体类的类名 -->
          <table tableName="t_emp" domainObjectName="Emp"/>
          <table tableName="t_dept" domainObjectName="Dept"/>
  
      </context>
  </generatorConfiguration>
  ```

- 执行MBG创建的generate目标

  ![image-20220506211728029](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/image-20220506211728029.png)

  

  ![image-20220506211839726](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/image-20220506211839726.png)

- QBC查询

  ```java
  package top.withlevi.mybatis.testmbg;
  
  import org.apache.ibatis.io.Resources;
  import org.apache.ibatis.session.SqlSession;
  import org.apache.ibatis.session.SqlSessionFactory;
  import org.apache.ibatis.session.SqlSessionFactoryBuilder;
  import org.junit.Test;
  import top.withlevi.mybatis.mapper.EmpMapper;
  import top.withlevi.mybatis.pojo.Emp;
  import top.withlevi.mybatis.pojo.EmpExample;
  
  import java.io.IOException;
  import java.io.InputStream;
  import java.util.List;
  
  /**
   * @author Levi.Zhao
   * @version 1.0
   * @date 2022/4/30 1:28 PM
   */
  public class TestMBG {
  
  
      @Test
      public void testMBG(){
          try {
              InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
              SqlSession sqlSession = sqlSessionFactory.openSession(true);
              EmpMapper mapper = sqlSession.getMapper(EmpMapper.class);
  
              // 查询所有数据
             /*  List<Emp> emps = mapper.selectByExample(null);
              emps.forEach(System.out::println);*/
  
              EmpExample empExample = new EmpExample();
              //empExample.createCriteria().andEmpNameEqualTo("张三").andAgeEqualTo(13);
              empExample.createCriteria().andEmpNameEqualTo("张三");
              empExample.or().andEmpNameIsNotNull();
              List<Emp> emps = mapper.selectByExample(empExample);
              emps.forEach(System.out::println);
  
  
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  
      @Test
      public void testMBG02(){
          try {
              InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
              SqlSession sqlSession = sqlSessionFactory.openSession(true);
              EmpMapper mapper = sqlSession.getMapper(EmpMapper.class);
              //mapper.updateByPrimaryKey(new Emp(1,"Levi",30,"男","levi@gmail.com","2"));
              mapper.updateByPrimaryKeySelective(new Emp(2,"王东晓",28,null,"wdx@gmail.com","1"));
  
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  }
  
  ```



### 十二、分页插件

#### 1、分页插件使用步骤

- 添加依赖

  ```xml
   <!-- https://mvnrepository.com/artifact/com.github.pagehelper/pagehelper -->
          <dependency>
              <groupId>com.github.pagehelper</groupId>
              <artifactId>pagehelper</artifactId>
              <version>5.2.0</version>
          </dependency>
  ```

- 配置分页插件

  在MyBatis的核心配置文件中配置插件

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?> <!DOCTYPE configuration
          PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
  <configuration>
  
      <!--引入外部数据库文件-->
      <properties resource="jdbc.properties"/>
  
      <!--设置别名-->
      <typeAliases>
          <package name="top.withlevi.mybatis.pojo"/>
      </typeAliases>
  
      <plugins>
          <!--设置分页插件-->
          <plugin interceptor="com.github.pagehelper.PageInterceptor"/>
      </plugins>
  
  
      <!--设置连接数据库的环境-->
      <environments default="development">
          <environment id="development">
              <transactionManager type="JDBC"/>
              <dataSource type="POOLED">
                  <property name="driver" value="${jdbc.driver}"/>
                  <property name="url"
                            value="${jdbc.url}"/>
                  <property name="username" value="${jdbc.username}"/>
                  <property name="password" value="${jdbc.password}"/>
              </dataSource>
          </environment>
      </environments>
  
      <!--引入映射文件-->
      <mappers>
          <!-- <mapper resource="UserMappers/UserMapper.xml"/>-->
          <package name="top.withlevi.mybatis.mapper"/>
      </mappers>
  </configuration>
  ```



#### 2、分页插件的使用

- 在查询功能之前使用PageHelper.startPage(int pageNum, int pageSize)开启分页功能

  > pageNum:当前页的页码
  > pageSize:每页显示的条数

- 在查询获取list集合之后，使用PageInfo<T> pageInfo = new PageInfo<>(List<T> list, int navigatePages)获取分页相关数据

  > list:分页之后的数据
  >
  > navigatePages:导航分页的页码数

- 分页相关数据

  > PageInfo{
  >  pageNum=8, pageSize=4, size=2, startRow=29, endRow=30, total=30, pages=8,
  >
  > list=Page{count=true, pageNum=8, pageSize=4, startRow=28, endRow=32, total=30, pages=8, reasonable=false, pageSizeZero=false},
  >
  > prePage=7, nextPage=0, isFirstPage=false, isLastPage=true, hasPreviousPage=true, hasNextPage=false, navigatePages=5, navigateFirstPage4, navigateLastPage8, navigatepageNums=[4, 5, 6, 7, 8]
  >
  > }
  >
  > 常用数据: pageNum:当前页的页码 
  >
  > pageSize:每页显示的条数 
  >
  > size:当前页显示的真实条数 
  >
  > total:总记录数 
  >
  > pages:总页数 p
  >
  > rePage:上一页的页码 
  >
  > nextPage:下一页的页码
  >
  > isFirstPage/isLastPage:是否为第一页/最后一页 
  >
  > hasPreviousPage/hasNextPage:是否存在上一页/下一页 
  >
  > navigatePages:导航分页的页码数 
  >
  > navigatepageNums:导航分页的页码，[1,2,3,4,5]

```java
package top.withlevi.mybatis.testmbg;

import com.github.pagehelper.Page;
import com.github.pagehelper.PageHelper;
import com.github.pagehelper.PageInfo;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;
import top.withlevi.mybatis.mapper.EmpMapper;
import top.withlevi.mybatis.pojo.Emp;

import java.io.IOException;
import java.io.InputStream;
import java.util.List;

/**
 * @author Levi.Zhao
 * @version 1.0
 * @date 2022/4/30 2:11 PM
 */
public class PageHelperTest {


    @Test
    public void testPageHelper(){
        try {
            InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
            SqlSession sqlSession = sqlSessionFactory.openSession(true);
            EmpMapper mapper = sqlSession.getMapper(EmpMapper.class);
            // Page<Object> pages = PageHelper.startPage(1, 3);
            PageHelper.startPage(1, 3);
            List<Emp> emps = mapper.selectByExample(null);
            PageInfo<Emp> pageInfo = new PageInfo<>(emps,3);
            System.out.println(pageInfo);
            // emps.forEach(System.out::println);

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```





