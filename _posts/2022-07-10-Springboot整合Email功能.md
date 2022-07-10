---
layout: post
title: Springboot整合Email功能
subtitle: spring-boot-starter-mail
date: 2022-07-10
author: Levi
header-img: img/springboot/post-bg-paypal.png
catalog: true
top: false
tags:
  - Springboot
  - Email

---



## 1、邮件发送：Mail

我们在注册很多的网站时，都会遇到邮件或是手机号验证，也就是通过你的邮箱或是手机短信去接受网站发给你的注册验证信息，填写验证码之后，就可以完成注册了，同时，网站也会绑定你的手机号或是邮箱。

那么，像这样的功能，我们如何实现呢？SpringBoot已经给我们提供了封装好的邮件模块使用：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

### 2、邮件相关技术背景

所有的用户都可以在电子邮件服务器上申请一个账号用于邮件发送和接收，那么邮件是以什么样的格式发送的呢？实际上和Http一样，邮件发送也有自己的协议，也就是约定邮件数据长啥样以及如何通信。

![img](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/851491-20161202143243756-1715308358.png)

比较常用的协议有两种：

1. SMTP协议（主要用于发送邮件 Simple Mail Transfer Protocol）
2. POP3协议（主要用于接收邮件 Post Office Protocol 3）

整个发送/接收流程大致如下：

![img](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/u=3675146129,445744702&fm=253&fmt=auto&app=138&f=JPG.jpeg)

实际上每个邮箱服务器都有一个smtp发送服务器和pop3接收服务器，比如要从QQ邮箱发送邮件到163邮箱，那么我们只需要通过QQ邮箱客户端告知QQ邮箱的smtp服务器我们需要发送邮件，以及邮件的相关信息，然后QQ邮箱的smtp服务器就会帮助我们发送到163邮箱的pop3服务器上，163邮箱会通过163邮箱客户端告知对应用户收到一封新邮件。

而我们如果想要实现给别人发送邮件，那么就需要连接到对应电子邮箱的smtp服务器上，并告知其我们要发送邮件。而SpringBoot已经帮助我们将最基本的底层通信全部实现了，我们只需要关心smtp服务器的地址以及我们要发送的邮件长啥样即可。

这里以qq邮箱 为例，我们需要在配置文件中告诉SpringBootMail我们的smtp服务器的地址以及你的邮箱账号和密码，首先我们要去设置中开启smtp/pop3服务才可以，开启后会得到一个随机生成的密钥，这个就是我们的密码。

```yaml
spring:
  mail:
  	# qq邮箱的地址为smtp.qq.com
    host: smtp.qq.com
    # 你申请的qq邮箱的地址
    username: zhangsan@qq.com
    # 开启smtp/pop3时自动生成的
    password: qdrxfcxhnpeebcnm
```



## 3、测试实现

1. 实现基本的邮件发送功能

```java
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessageHelper;

import javax.annotation.Resource;
import javax.mail.MessagingException;
import javax.mail.internet.MimeMessage;
import java.io.File;

@SpringBootTest
class SpringbootEmailApplicationTests {

	// JavaMailSender 是专门用于发送邮件的对象,自动配置类已经提供了Bean
	@Resource
	JavaMailSender sender;

	@Test
	void contextLoads() throws MessagingException {
      	// SimpleMailMessage是一个比较简易的邮件封装,支持设置一些比较简单的内容
		SimpleMailMessage message = new SimpleMailMessage();
		// 设置邮件主题
		message.setSubject("[利瓦伊科技有限公司-Offer Letter]");
		// 设置邮件内容
		message.setText("小米椒同学您好，经过多轮面试，恭喜您被我们录取了，期待咱们未来一起工作.");
		// 设置收件人邮箱地址
		message.setTo("xxxxx@126.com");
		// 设置发件人邮箱地址
		message.setFrom("zhangsanqq.com");
		// 发送邮件
		sender.send(message);
	}
}

```



2. 实现复杂的邮件功能

```java
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessageHelper;

import javax.annotation.Resource;
import javax.mail.MessagingException;
import javax.mail.internet.MimeMessage;
import java.io.File;

@SpringBootTest
class SpringbootEmailApplicationTests {

	// JavaMailSender 是专门用于发送邮件的对象,自动配置类已经提供了Bean
	@Resource
	JavaMailSender sender;

	@Test
	void contextLoads() throws MessagingException {
    // 创建一个MimeMessage
    MimeMessage message = sender.createMimeMessage();
    // 使用MimeMessageHelper来帮助我们修改MimeMessage中的信息
		MimeMessageHelper helper = new MimeMessageHelper(message, true);
    // 设置邮件主题
		helper.setSubject("[使用MimeMessage来发送邮件]");
    // 设置邮件内容
		helper.setText("如果需要添加附件等更多功能，可以使用MimeMessageHelper来帮助我们完成");
    // 设置邮件附件
		helper.addAttachment("kuangshen.pdf",new File("/Users/xxxxx/Desktop/code/30、整合		     Dubbo+Zookeeper.pdf"));
    // 设置收件人邮箱
		helper.setTo("xxxxx@126.com");
    // 设置抄送人的邮箱
		helper.setCc("xxxxxx@gmail.com");
    // 设置密送人的邮箱
		helper.setBcc("xxxxxx@apple.com");
    // 设置发件人的邮箱
		helper.setFrom("zhangsan@qq.com");
		// 发送修改好的内容
		sender.send(message);
	}
}
```



