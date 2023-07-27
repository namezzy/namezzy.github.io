---
layout: post
title: MiddleWare-RabbitMQ
subtitle: RabbitMQ
date: 2022-09-23
author: Levi
header-img: img/rabbitmq/rabbi1tmq.jpg
catalog: true
top: false
tags:
  - RabbitMQ
  - Channel
  - Exchange
  - Virtual Host
---

![image-20220415163559986](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/202209221601898.jpeg)

##  RabbitMQ

我们之前如果需要进行远程调用，那么一般可以通过发送HTTP请求来完成，而现在，我们可以使用第二种方式，就是消息队列，它能够将发送方发送的信息放入队列中，当新的消息入队时，会通知接收方进行处理，一般消息发送方称为生产者，接收方称为消费者。



![image-20220415165805716](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/202209221601371.jpeg)



这样我们所有的请求，都可以直接丢到消息队列中，再由消费者取出，不再是直接连接消费者的形式了，而是加了一个中间商，这也是一种很好的解耦方案，并且在高并发的情况下，由于消费者能力有限，消息队列也能起到一个削峰填谷的作用，堆积一部分的请求，再由消费者来慢慢处理，而不会像直接调用那样请求蜂拥而至。

那么，消息队列具体实现有哪些呢：

* RabbitMQ  -  性能很强，吞吐量很高，支持多种协议，集群化，消息的可靠执行特性等优势，很适合企业的开发。
* Kafka - 提供了超高的吞吐量，ms级别的延迟，极高的可用性以及可靠性，而且分布式可以任意扩展。
* RocketMQ  -  阿里巴巴推出的消息队列，经历过双十一的考验，单机吞吐量高，消息的高可靠性，扩展性强，支持事务等，但是功能不够完整，语言支持性较差。

我们这里，主要讲解的是RabbitMQ消息队列。



### RabbitMQ 消息队列

**官方网站：**https://www.rabbitmq.com

> RabbitMQ拥有数万计的用户，是最受欢迎的开源消息队列之一，从[T-Mobile](https://www.youtube.com/watch?v=1qcTu2QUtrU)到[Runtastic](https://medium.com/@runtastic/messagebus-handling-dead-letters-in-rabbitmq-using-a-dead-letter-exchange-f070699b952b)，RabbitMQ在全球范围内用于小型初创企业和大型企业。
>
> RabbitMQ轻量级，易于在本地和云端部署，它支持多种消息协议。RabbitMQ可以部署在分布式和联合配置中，以满足大规模、高可用性要求。
>
> RabbitMQ在许多操作系统和云环境中运行，并为[大多数流行语言](https://www.rabbitmq.com/devtools.html)提供了[广泛的开发者工具](https://www.rabbitmq.com/devtools.html)。

### 安装消息队列

在这里我使用docker进行安装部署

```shell
# 云服务器 提前开始端口  15672: web访问  5672：通信端口  4369：集群通信  25672：集群通信

docker run -d --hostname rabbitmq04 --name rabbitmq04 -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin -p 15672:15672 -p 5672:5672 -p4369:4369 -p25672:25672 macintoshplus/rabbitmq-management
```

安装完成之后，查看安装状态

```shell
docker ps

PS C:\Users\Hello World> docker ps
CONTAINER ID   IMAGE                               COMMAND                  CREATED              STATUS              PORTS                                                                                                                     NAMES
2b4a1ca1470d   macintoshplus/rabbitmq-management   "/docker-entrypoint.…"   About a minute ago   Up About a minute   0.0.0.0:4369->4369/tcp, 5671/tcp, 0.0.0.0:5672->5672/tcp, 0.0.0.0:15672->15672/tcp, 0.0.0.0:25672->25672/tcp, 15671/tcp   rabbitmq04
```

进入RabbitMQ中查看运行状态

```shell
# 先进入rabbitmq容器内部
docker exec -it rabbitmq04 /bin/bash

# 查看运行状态
rabbitmqctl status

root@rabbitmq04:/# rabbitmqctl status
Status of node rabbit@rabbitmq04 ...
[{pid,253},
 {running_applications,
     [{rabbitmq_management,"RabbitMQ Management Console","3.6.2"},
      {rabbitmq_web_dispatch,"RabbitMQ Web Dispatcher","3.6.2"},
      {rabbitmq_management_agent,"RabbitMQ Management Agent","3.6.2"},
      {rabbit,"RabbitMQ","3.6.2"},
      {mnesia,"MNESIA  CXC 138 12","4.13.3"},
      {os_mon,"CPO  CXC 138 46","2.4"},
      {amqp_client,"RabbitMQ AMQP Client","3.6.2"},
      {webmachine,"webmachine","1.10.3"},
      {mochiweb,"MochiMedia Web Server","2.13.1"},
      {compiler,"ERTS  CXC 138 10","6.0.3"},
      {ssl,"Erlang/OTP SSL application","7.3"},
      {public_key,"Public key infrastructure","1.1.1"},
      {crypto,"CRYPTO","3.6.3"},
      {asn1,"The Erlang ASN1 compiler version 4.0.2","4.0.2"},
      {rabbit_common,[],"3.6.2"},
      {ranch,"Socket acceptor pool for TCP protocols.","1.2.1"},
      {syntax_tools,"Syntax tools","1.7"},
      {inets,"INETS  CXC 138 49","6.2"},
      {xmerl,"XML parser","1.3.10"},
      {sasl,"SASL  CXC 138 11","2.7"},
      {stdlib,"ERTS  CXC 138 10","2.8"},
      {kernel,"ERTS  CXC 138 10","4.2"}]},
 {os,{unix,linux}},
 {erlang_version,
     "Erlang/OTP 18 [erts-7.3] [source] [64-bit] [smp:8:8] [async-threads:128] [kernel-poll:true]\n"},
 {memory,
     [{total,154787648},
      {connection_readers,0},
      {connection_writers,0},
      {connection_channels,0},
      {connection_other,2712},
      {queue_procs,2712},
      {queue_slave_procs,0},
      {plugins,460800},
      {other_proc,18757576},
      {mnesia,67128},
      {mgmt_db,265040},
      {msg_index,46736},
      {other_ets,1415832},
      {binary,29328},
      {code,27709928},
      {atom,992409},
      {other_system,105037447}]},
 {alarms,[]},
 {listeners,[{clustering,25672,"::"},{amqp,5672,"::"}]},
 {vm_memory_high_watermark,0.4},
 {vm_memory_limit,5275194163},
 {disk_free_limit,50000000},
 {disk_free,250525818880},
 {file_descriptors,
     [{total_limit,1048476},
      {total_used,2},
      {sockets_limit,943626},
      {sockets_used,0}]},
 {processes,[{limit,1048576},{used,229}]},
 {run_queue,0},
 {uptime,203},
 {kernel,{net_ticktime,60}}]
root@rabbitmq04:/#
```

登录web端查看管理界面

```shell
# 安装Rabbitmq机器的ip地址：15672
172.30.210.68:15672
```

![image-20220922161623988](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/202209221616094.png)

进入之后会显示当前的消息队列情况, 包括版本号，Erlang版本等，这里需要介绍一下RabbitMQ的设计架构

![image-20220416103043845](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/202209221618371.jpeg)



* **生产者（Publisher）和消费者（Consumer）：**不用多说了吧。
* **Channel：**我们的客户端连接都会使用一个Channel，再通过Channel去访问到RabbitMQ服务器，注意通信协议不是http，而是amqp协议。
* **Exchange：**类似于交换机一样的存在，会根据我们的请求，转发给相应的消息队列，每个队列都可以绑定到Exchange上，这样Exchange就可以将数据转发给队列了，可以存在很多个，不同的Exchange类型可以用于实现不同消息的模式。
* **Queue：**消息队列本体，生产者所有的消息都存放在消息队列中，等待消费者取出。
* **Virtual Host：**有点类似于环境隔离，不同环境都可以单独配置一个Virtual Host，每个Virtual Host可以包含很多个Exchange和Queue，每个Virtual Host相互之间不影响。



### 使用消息队列

我们就从最简的的模型开始讲起：

![image-20220417103647609](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/202209221621264.jpeg)

（一个生产者 -> 消息队列 -> 一个消费者）

生产者只需要将数据丢进消息队列，而消费者只需要将数据从消息队列中取出，这样就实现了生产者和消费者的消息交互。我们现在来演示一下，首先进入到我们的管理页面，这里我们创建一个新的实验环境，只需要新建一个Virtual Host即可：



![image-20220922162353779](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/202209221623853.png)

添加新的虚拟主机之后，我们可以看到，当前admin用户的主机访问权限中新增了我们刚刚添加的环境

![image-20220922162839402](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/202209221628473.png)

![image-20220922163000734](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/202209221630803.png)

现在我们来看看交换机:

![image-20220922163051948](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/202209221630033.png)



交换机列表中自动为我们新增了刚刚创建好的虚拟主机相关的预设交换机，一共7个，这里我们首先介绍一下前面两个`direct`类型的交换机，一个是`（AMQP default）`还有一个是`amq.direct`，它们都是直连模式的交换机，我们来看看第一个：

![image-20220922163230677](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/202209221632753.png)



第一个交换机是所有虚拟主机都会自带的一个默认交换机，并且此交换机不可删除，此交换机默认绑定到所有的消息队列，如果是通过默认交换机发送消息，那么会根据消息的`routingKey`（之后我们发消息都会指定）决定发送给哪个同名的消息队列，同时也不能显示地将消息队列绑定或解绑到此交换机。

我们可以看到，详细信息中，当前交换机特性是持久化的，也就是说就算机器重启，那么此交换机也会保留，如果不是持久化，那么一旦重启就会消失。实际上我们在列表中看到`D`的字样，就表示此交换机是持久化的，包含一会我们要讲解的消息队列列表也是这样，所有自动生成的交换机都是持久化的。

我们接着来看第二个交换机，这个交换机是一个普通的直连交换机：

![image-20220922163431628](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/202209221634702.png)



这个交换机和我们刚刚介绍的默认交换机类型一致，并且也是持久化的，但是我们可以看到它是具有绑定关系的，如果没有指定的消息队列绑定到此交换机上，那么这个交换机无法正常将信息存放到指定的消息队列中，也是根据`routingKey`寻找消息队列（但是可以自定义）

我们可以在下面直接操作，让某个队列绑定，这里我们先不进行操作。

介绍完了两个最基本的交换机之后（其他类型的交换机我们会在后面进行介绍），我们接着来看消息队列：

![image-20220922163548932](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/202209221635013.png)

可以看到消息队列列表中没有任何的消息队列，我们可以来尝试添加一个新的消息队列：

![image-20220922163646326](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/202209221636384.png)



第一行，我们选择我们刚刚创建好的虚拟主机，在这个虚拟主机下创建此消息队列，接着我们将其类型定义为`Classic`类型，也就是经典类型（其他类型我们会在后面逐步介绍）名称随便起一个，然后持久化我们选择`Transient`暂时的（当然也可以持久化，看你自己）自动删除我们选择`No`（需要至少有一个消费者连接到这个队列，之后，一旦所有与这个队列连接的消费者都断开时，就会自动删除此队列）最下面的参数我们暂时不进行任何设置（之后会用到）

现在，我们就创建好了一个经典的消息队列：

![image-20220922163733457](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/202209221637502.png)

点击此队列的名称，我们可以查看详细信息：

![image-20220922163812345](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/202209221638415.png)

详细信息中包括队列的当前负载状态、属性、消息队列占用的内存，消息数量等，一会我们发送消息时可以进一步进行观察。

现在我们需要将此消息队列绑定到上面的第二个直连交换机，这样我们就可以通过此交换机向此消息队列发送消息了：

![image-20220922163935291](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/202209221639354.png)

这里填写之前第二个交换机的名称还有我们自定义的`routingKey`（最好还是和消息队列名称一致，这里是为了一会演示两个交换机区别用）我们直接点击绑定即可：

绑定之后我们可以看到当前队列已经绑定对应的交换机了，现在我们可以前往交换机对此消息队列发送一个消息：

![image-20220922164008804](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/202209221640858.png)



回到交换机之后，可以卡到这边也是同步了当前的绑定信息，在下方，我们直接向此消息队列发送信息：

![image-20220923144943837](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/202209231449923.png)

点击发送之后，我们回到刚刚的交换机详细页面，可以看到已经有一条新的消息在队列中了：

![image-20220923145138391](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/202209231451453.png)

我们可以直接在消息队列这边获取消息队列中的消息，找到下方的Get message选项：

获取发送的消息

![image-20220923145234213](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/202209231452303.png)

当然除了在交换机发送消息和消息队列之外，我们也可以直接在消息队列这里发：

![image-20220923145419610](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/202209231454687.png)

效果是一样的，注意这里我们可以选择是否将消息持久化，如果是持久化消息，那么就算服务器重启，此消息也会保存在消息队列中。

最后如果我们不需要再使用此消息队列了，我们可以手动对其进行删除或是清空：

![image-20220923145519988](https://cdn.jsdelivr.net/gh/Levi0219/note-photo/202209231455042.png)

点击Delete Queue删除我们刚刚创建好的`yyds`队列，到这里，我们对消息队列的一些简单使用记录展示完毕。

