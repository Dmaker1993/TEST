---
title: RabbitMQ基础与集群实战
top: false
date: 2021-12-23 18:10:32
tags:
    - 中间件
    - 集群
    - RabbitMQ
categories:
    - 中间件
---
# RabbitMQ实战教程

> 视频：https://www.bilibili.com/video/BV1dE411K7MG?from=search&seid=15593601763323732951&spm_id_from=333.337.0.0

## 1.MQ引言

### 1.1 什么是MQ

​		`MQ`(Message Queue)：翻译为`消息队列`，通过典型的`生产者`和`消费者`模型，生产者不断向消息队列中生产消息，消费者不断从队列中获取消息。因为消息的生产和消费都是异步的，而且只关心消息的消费和生产，没有业务逻辑的侵入，轻松的实现系统间的解藕。别名为：`消息中间件`，通过利用高效可靠的消息传递机制进行平台无关的数据交流，并基于数据通信来进行分布式系统的集成。

### 1.2 MQ有哪些

​		当今市面上有很多的消息中间件，如老牌的`ActiveMQ`，`RabbitMQ`，炙手可热的`Kafka`、阿里巴巴自主研发的`RocketMQ`等。

### 1.3 不同MQ的特点

```markdown
# 1.ActiveMQ
	ActiveMQ是Apache出品，最流行的，能力强劲的开源消息总线。他是一个完全支持JMS规范的消息中间件。丰富的API，多种集群架构模式让ActiveMQ称为老牌的消息中间件，在中小企业颇受欢迎。
# 2.Kafka
	Kafka是LinkedIn公司开源的分布式发布-订阅消息中间件，目前属于Apache顶级项目。Kafka的主要特点是基于pull模式来处理消息消费。追求高吞吐量，一开始的目的是用于日志的收集和传输。0.8版本开始支持复制，不支持事物，对消息的丢失、重复、错误没有严格的要求。适合产生大量数据的互联网服务的数据收集业务。
# 3.RocketMQ
	RocketMQ是阿里开源的消息中间件。他是纯java开发，具有高吞吐量、高可用性、适合大规模分布式系统应用的特点。RocketMQ的思路起源于Kafka，但并不是Kafka的一个复制，他对消息的可靠传输及事物做了优化，目前的阿里集团被广泛应用于交易、充值、流计算、消息推送、日志流处理、binglog分发等场景。
# 4.RabbitMQ
	RabbitMQ是使用Erlang语言开发的开源消息中间件系统，基于AMQP协议来实现。AMQP的主要特征是面向消息、队列、路由（包括点对点和发布订阅）、可靠性、安全。AMQP协议更多用在企业内对数据一致性、稳定性和可靠性要求很高的场景，对性能和吞吐量的要求还在其次。
```

> RabbitMQ比Kafka可靠，Kafka更适合IO高吞吐的处理，一般应用在大数据日志处理或对实时性（延时低）、可靠性（少量丢数据）要求稍低的场景使用，比如ELK日志收集。

## 2.RabbitMQ的引言

### 2.1 RabbitMQ简介

​		RabbitMQ是基于AMQP协议，erlang语言开发，是部署最广泛的开源消息中间件，也是最受欢迎的消息中间件之一。

![image](https://img-blog.csdnimg.cn/681db2b5bf6f486f8dfa258fa90aad53.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

```markdown
# AMQP协议（后续单独讲）
	AMQP(Advanced Message Queuing Protocol，高级消息队列协议)，在2003年被提出，在最用于解决金融领域不同平台之间消息传递的交互问题。顾名思义，AMQP是一种协议，更准确的说是一种binary wrie-level protocol（链接协议）。这时其和JMS的本质差异，AMQP不从api层进行限定，而是直接定义网络交换的数据格式。这使得实现了AMQP的provider天然性就是跨平台的。
								Server
					+---------------------------+
					|		Virtual Host		|
					|	+--------------------+  |
   	+------------+  |	|	+-----------+    |	|
	| Publisher  | -------->| Exchange	|    |  |
	| Application|	|	|	+-----+-----+    |  |
	+------------+	|   | 		  |			 |	|
					|	|	+-----------+	 |	|	+-------------+
					|	|	+  Message	+	 |	|	|  Consummer  |
					|	|	+	Queue	+ --------->| Application |
					|	|	+-----------+	 |	|	+-------------+
					|	+--------------------+	|
					+---------------------------+
```

### 2.2 RabbitMQ的安装

​		本次安装的环境如下：

- 系统：centos8 64位
- erlang：24.1.7
- rabbitmq：3.9.11

#### 2.2.1 erlang下载

> 因为rabbitMQ是基于erlang开发的，所以先要下载erlang的包：https://github.com/rabbitmq/erlang-rpm
>
> erlang版本和rabbitmq版本对照：https://www.rabbitmq.com/which-erlang.html

![iamge](https://img-blog.csdnimg.cn/bf7aa8f5462148a99f279b03dcb493d1.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

​		本次开发使用的最新的rabbitmq版本为`3.9.11`，最小支持的erlang版本为23.2，所以本次erlang使用了`24.1.7`版本

![iamge](https://img-blog.csdnimg.cn/889a43dfbe3048b58c0622101762bbad.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

#### 2.2.2 RabbitMQ下载

> rabbitmq下载地址：https://www.rabbitmq.com/install-rpm.html#downloads


![image](https://img-blog.csdnimg.cn/40404dec0e524d10bb9df1a1d16f9b5d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

#### 2.2.3 安装启动

```markdown
# 1.将rabbitmq相关包上传到linxu服务器中
	使用scp命令上传到服务器两个包：
	scp ./erlang-24.1.7-1.el8.x86_64.rpm root@10.3.4.5:/root/
	scp ./rabbitmq-server-3.9.11-1.el8.noarch.rpm root@10.3.4.5:/root/
# 2.依次安装erlang、rabbitmq
	使用rpm命令进行安装，缺少依赖会进行提示：
	rpm -ivh erlang-24.1.7-1.el8.x86_64.rpm
	rpm -ivn rabbitmq-server-3.9.11-1.el8.noarch.rpm
# 3.修改rabbitmq的配置
	使用rpm包的方式安装时，并没有将配置文件也放到指定目录下，所以需要自行创建一个配置文件：/etc/rabbitmq/rabbitmq.conf，具体的配置内容可以参考：https://www.rabbitmq.com/configure.html#config-file-formats。
	这里我们需要修改一处：loopback_users=none，表示能够让guest用户进行远程访问。默认情况下，guest用户只能在localhost域名下访问。我们使用的是云服务器，需要使用ip进行访问，所以需要修改这个配置。
# 4.开启管理控制台插件
	其实就是开启rabbitmq的一个插件：rabbitmq_management，可以让我们使用web界面管理rabbitmq。执行命令：rabbitmq-plugins enable rabbitmq_management。该命令还另外开启了2个插件：rabbitmq_management_agent、rabbitmq_web_dispatch。
# 5.启动/停止/重启rabbitmq服务
	rabbitmq安装的时候，会将其设置为系统服务，使用系统服务命令即可：
	systemctl start/stop/restart rabbitmq-server.service
# 6.查看服务状态
	systemctl stauts rabbitmq-server.service
    结果如下为正常运行中：
    ● rabbitmq-server.service - RabbitMQ broker
       Loaded: loaded (/usr/lib/systemd/system/rabbitmq-server.service; disabled; vendor preset: disabled)
       Active: active (running) since Sun 2021-12-12 23:56:51 CST; 1 day 9h ago
      Process: 22710 ExecStop=/usr/sbin/rabbitmqctl shutdown (code=exited, status=0/SUCCESS)
     Main PID: 22758 (beam.smp)
        Tasks: 23 (limit: 23722)
       Memory: 94.8M
       CGroup: /system.slice/rabbitmq-server.service
               ├─22758 /usr/lib64/erlang/erts-12.1.5/bin/beam.smp -W w -MBas ageffcbf -MHas ageffcbf -MBlmbcs 512 -MHlmbcs 512 -MMmcs 30 -P 1048576 -t 5000000 -stbt db -zdbbl 12800>
               ├─22773 erl_child_setup 32768
               ├─22827 inet_gethost 4
               └─22828 inet_gethost 4
# 7.访问管理界面
 	默认http后台界面的端口为15672。
 	http://10.3.4.5:15672/
# 8.登陆
	账号/密码：guest/guest
```

## 3.RabbitMQ配置

### 3.1 RabbitMQ管理命令行

```markdown
# 1.服务管理
	systemctl start/stop/restart rabbitmq-server.service
# 2.管理命令行
	可以用来在不使用web管理端的情况下管理rabbitmq。
	rabbitmqctl help // 查看所有的命令
# 3.插件管理
	rabbitmq-plugins enable/list/disable
```

### 3.2 web管理介绍

![image](https://img-blog.csdnimg.cn/6a2017fe4a3a4216a594f1d64c3a28e6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

- **Connection**：连接，无论是消费者还是生产者，都要与rabbitmq建立连接才能进行消息的生产与消费。
- **Channels**：通道，建立连接后，消息的投递和获取都是通过通道来进行的。
- **Exchages**：交换机，用来实现消息的路有。
- **Queues**：队列，消息存放于该队列中，等待消费，消费之后移除。

#### 3.2.1用户和虚拟主机管理

##### 3.2.1.1 用户

![image](https://img-blog.csdnimg.cn/67b980ab25e24c75b2a6d3c9590c22ff.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

```markdown
# Tags说明：
	Admin(超级管理员)：登录控制台，查看所有信息，并对用户、策略（policy）进行修改。
	Monitoring（监控者）：登陆控制台，查看rabbitmq的节点（进程数、内存、磁盘等使用情况）信息。
	Policymaker（策略制定者）：登陆控制台，对policy进行管理。
	Management（普通管理员）：登陆控制台查看信息。
	其他：无法登陆控制台，就是普通的消费者或者生产者。
```

##### 3.2.1.2 虚拟主机

![image](https://img-blog.csdnimg.cn/b21620fc59f5467ebc4c0402d5e2c0df.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

```markdown
# 虚拟主机：
	为了让各个用户可以互不干扰的工作，RabbitMQ添加了虚拟主机（Virtual Host）概念。其实就是一个独立的访问路径，不同用户使用不同路径，各自都有自己的队列、交换机，互不影响。相当于MySQL中的数据库。
```

##### 3.2.1.3 用户和虚拟主机绑定

![image](https://img-blog.csdnimg.cn/e60681b36bec47b5bba1ebde7ee8d03b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

![image](https://img-blog.csdnimg.cn/b4808afbd6e34a8e982624a3f0b02856.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

## 4.RabbitMQ的Java客户端

### 4.1 AMQP协议回顾

![image](https://img-blog.csdnimg.cn/eb827e01a84b4eea9bddb6dab2947c84.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

### 4.2 RabbitMQ支持的消息类型

![image](https://img-blog.csdnimg.cn/40d40399960442e9a2d0d5668a6825d2.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

### 4.3 引入依赖

```xml
<dependency>
  <groupId>com.rabbitmq</groupId>
  <artifactId>amqp-client</artifactId>
  <version>5.14.0</version>
</dependency>
```

### 4.4 各种模型客户端代码

#### 4.4.1 Direct-直连模式

![image](https://img-blog.csdnimg.cn/2401e687d01347fb9827dffb97b570c9.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_18,color_FFFFFF,t_70,g_se,x_16#pic_center)

```markdown
# 相关概念：
	P：provider，生产者，要发送消息的程序
	C：consumer，消费者，消费消息的程序
	Queue：队列，存储消息的地方
```

**Producer：**

```java
/**
 * direct模式-生产者
 * <p>
 * 直接点对点，也就是生产者将消息发送到队列中，消费者直接从队列中获取
 *
 * @author secret
 * @date 2021/12/14 11:34 AM
 */
public class TestProducer {
    // 定义队列名字
    public static final String QUEUE_NAME = "hello";

    public static void main(String[] args) throws IOException, TimeoutException {
        // 1.创建连接工程
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("101.43.52.186");
        connectionFactory.setPort(5672);
        connectionFactory.setVirtualHost("/adu");
        connectionFactory.setUsername("adu");
        connectionFactory.setPassword("adu");
        // 2.创建连接
        Connection connection = connectionFactory.newConnection();
        // 3.创建通道
        Channel channel = connection.createChannel();
        // 4.声明队列：队列名称，是否持久化，是否独占，额外参数
        channel.queueDeclare(QUEUE_NAME, true, false, false, null);
        // 5.向队列发送消息：交换机名称，队列名称，额外参数，消息体
        channel.basicPublish("", QUEUE_NAME, null, "hello world".getBytes(StandardCharsets.UTF_8));
        // 6.关闭资源
        channel.close();
        connection.close();
    }
}
```

**Consumer**:

```java
/**
 * direct模式-消费者
 *
 * @author secret
 * @date 2021/12/14 11:34 AM
 */
public class TestConsumer {
    public static final String QUEUE_NAME = "hello";

    public static void main(String[] args) throws IOException, TimeoutException {
        // 1 创建连接工厂，并设置基础参数
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("101.43.52.186");
        connectionFactory.setPort(5672);
        connectionFactory.setVirtualHost("/adu");
        connectionFactory.setUsername("adu");
        connectionFactory.setPassword("adu");
        // 2 创建连接
        Connection connection = connectionFactory.newConnection();
        // 3 创建通道
        Channel channel = connection.createChannel();
        // 4 声明队列
        channel.queueDeclare(QUEUE_NAME, true, false, false, null);
        // 4 接收消息
        channel.basicConsume(QUEUE_NAME, true, new DefaultConsumer(channel) {
            // 回调方法
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("consumer：" + new String(body, StandardCharsets.UTF_8));
            }
        });
        // 5 关闭资源
        channel.close();
        connection.close();
    }
}
```

#### 4.4.2 Work Queues-工作队列

![image](https://img-blog.csdnimg.cn/37c7326912a04ff1baaa25754a61535f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

​		Work Queue也被称为`Task Queue`任务模型。当消息处理比较耗时时，可能消息生产的速度远远超过了消息消费的速度。长此以往，消息就会堆积，此时就可以使用Work Queue模型。让多个消费者绑定同一个队列，共同消费队列中的消息。队列中的消息一旦被消费，就会被删除，因此不会产生重复消费。

```java
// 提取连接rabbitmq的工具类
public class RabbitMQUtil {

    public static Connection getConnection()    {
        try {
            // 创建连接工厂
            ConnectionFactory connectionFactory = new ConnectionFactory();
            // 设置参数
            connectionFactory.setHost("101.43.52.186");
            connectionFactory.setPort(5672);
            connectionFactory.setVirtualHost("/adu");
            connectionFactory.setUsername("adu");
            connectionFactory.setPassword("adu");
            // 创建连接
            return connectionFactory.newConnection();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (TimeoutException e) {
            e.printStackTrace();
        }
        return null;
    }

    public static Channel getChannel(Connection connection) {
        try {
            return connection.createChannel();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }

    public static void close(Channel channel, Connection connection) {
        if (channel != null) {
            try {
                channel.close();
            } catch (IOException e) {
                e.printStackTrace();
            } catch (TimeoutException e) {
                e.printStackTrace();
            }
        }
        if (connection != null) {
            try {
                connection.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

**Producer：**

```java
/**
 * Work Queue模型
 * <p>
 * 工作队列模型，应对的是消费者消费比较慢的情况，多个消费者共同消费同一个队列中的消息。
 *
 * @author secret
 * @date 2021/12/14 2:02 PM
 */
public class WorkQueueProducer {

    public static final String QUEUE_NAME = "work";

    public static void main(String[] args) throws IOException {
        // 获取连接
        Connection connection = RabbitMQUtil.getConnection();
        // 获取通道
        Channel channel = RabbitMQUtil.getChannel(connection);
        // 声明队列
        channel.queueDeclare(QUEUE_NAME, true, false, false, null);
        // 发消息
        for (int i = 0; i < 10; i++) {
            channel.basicPublish("", QUEUE_NAME, null, ("hello work queue " + i).getBytes(StandardCharsets.UTF_8));
        }
        // 关闭资源
        RabbitMQUtil.close(channel, connection);
    }
}
```

**Consumer：**

```java
// consumer1
public class WorkQueueConsumer1 {
    public static final String QUEUE_NAME = "work";
    public static void main(String[] args) throws IOException {
        // 获取连接
        Connection connection = RabbitMQUtil.getConnection();
        // 获取管道
        Channel channel = RabbitMQUtil.getChannel(connection);
        // 声明队列
        channel.queueDeclare(QUEUE_NAME, true, false, false, null);
        // 消费消息
        channel.basicConsume(QUEUE_NAME, true, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("consumer1: " + new String(body, StandardCharsets.UTF_8));
            }
        });
    }
}

// consumer2
和consumer1相同代码，修改消费输出：System.out.println("consumer2: " + new String(body, StandardCharsets.UTF_8));
```

**结果：**

```markdown
# consumer1 output...
consumer1: hello work queue 0
consumer1: hello work queue 2
consumer1: hello work queue 4
consumer1: hello work queue 6
consumer1: hello work queue 8
# consumer2 output...
consumer2: hello work queue 1
consumer2: hello work queue 3
consumer2: hello work queue 5
consumer2: hello work queue 7
consumer2: hello work queue 9
```

**总结：**

​		默认情况下，RabbitMQ将按顺序依次将消息发给每个消费者，也就是说每个消费者都会收到相同数量的消息，这种平分消息的方式称为`循环`。

##### 4.4.2.1 消息自动确认机制

​		如果一个消费者在执行任务的过程中宕机，那么在以上代码中，一旦RabbitMQ将将消息传递给消费者，他就会立即标记为删除，那么该消费者中未处理的消息将丢失。我们希望在处理任务的过程中，一个消费者宕机了，能够将任务交给其他消费者处理。这时候我们就可以用到`消息自动确认机制`。也就是说，我们可以在处理完任务之后，再通知RabbitMQ可以删除该任务。

**Producer：**

```java
// 同上
```

**Consumer：**

```java
// consumer1
public class WorkQueueConsumer3 {
    public static final String QUEUE_NAME = "work";
    public static void main(String[] args) throws IOException {
        // 获取连接
        Connection connection = RabbitMQUtil.getConnection();
        // 获取管道
        Channel channel = RabbitMQUtil.getChannel(connection);
        // 声明队列
        channel.queueDeclare(QUEUE_NAME, true, false, false, null);
        // 每次只能消费一个消息
        channel.basicQos(1);
        // 消费消息
        channel.basicConsume(QUEUE_NAME, false, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                try {
                    Thread.sleep(1000); // 模拟处理消息消耗时常
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("consumer1: " + new String(body, StandardCharsets.UTF_8));
                // 手动确认，参数1：消息标记 参数2：是否同时确认多条，false表示每次只能确认一条
                channel.basicAck(envelope.getDeliveryTag(), false);
            }
        });
    }

}
// consumer2
public class WorkQueueConsumer4 {
    public static final String QUEUE_NAME = "work";
    public static void main(String[] args) throws IOException {
        // 获取连接
        Connection connection = RabbitMQUtil.getConnection();
        // 获取管道
        Channel channel = RabbitMQUtil.getChannel(connection);
        // 声明队列
        channel.queueDeclare(QUEUE_NAME, true, false, false, null);
        // 限制每次消费1条
        channel.basicQos(1);
        // 消费消息，autoAck改为false，表示不自动确认
        channel.basicConsume(QUEUE_NAME, false, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                try {
                    Thread.sleep(1500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("consumer1: " + new String(body, StandardCharsets.UTF_8));
                // 手动确认
                channel.basicAck(envelope.getDeliveryTag(), false);
            }
        });
    }
}
```

**结果：**

```markdown
# consumer1 output...
consumer1: hello work queue 1
consumer1: hello work queue 3
consumer1: hello work queue 5
consumer1: hello work queue 8
consumer1: hello work queue 10
consumer1: hello work queue 13
consumer1: hello work queue 15
consumer1: hello work queue 18
# consumer2 output...
consumer1: hello work queue 0
consumer1: hello work queue 2
consumer1: hello work queue 4
consumer1: hello work queue 6
consumer1: hello work queue 7
consumer1: hello work queue 9
consumer1: hello work queue 11
consumer1: hello work queue 12
consumer1: hello work queue 14
consumer1: hello work queue 16
consumer1: hello work queue 17
consumer1: hello work queue 19
```

**总结：**

​		实现消息自动确认，需要注意两点：1）设置通道一次只能消费一条消息 2）关闭自动确认，手动确认消息

#### 4.4.3 Publish/Subscribe-发布订阅模式

![image](https://img-blog.csdnimg.cn/610277d029c04fe5a3933cf79f21dcc2.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

```markdown
# 消息流程：
	可以有多个消费者，每个消费者都有自己的queue（队列），每个队列都要绑定要exchange（交换机）。
	生产者发送的消息只能发送给交换机，交换机来决定发送给哪个队列，生产者无法决定。
	交换机把消息发送给绑定过的所有队列，队列的消费者都能拿到消息，实现一个消息被多个消费者消费。
# 对应交换机类型：fanout
```

**Proudcer：**

```java
public class PublishSubscriptProducer {

    public static final String QUEUE_NAME = "fanout";

    public static void main(String[] args) throws IOException {
        // 获取连接
        Connection connection = RabbitMQUtil.getConnection();
        // 获取通道
        Channel channel = RabbitMQUtil.getChannel(connection);
        // 声明交换机 交换机名称 交换机类型
        channel.exchangeDeclare("logs", "fanout");
        // 发送消息到交换机
        channel.basicPublish("logs", "", null, "hello fanout".getBytes(StandardCharsets.UTF_8));
        // 关闭资源
        RabbitMQUtil.close(channel, connection);
    }
}
```

**Consumer：**

```java
// consumer1
public class PublishSubscribeConsumer1 {

    public static void main(String[] args) throws IOException {
        // 获取连接
        Connection connection = RabbitMQUtil.getConnection();
        // 获取通道
        Channel channel = RabbitMQUtil.getChannel(connection);
        // 声明交换机
        channel.exchangeDeclare("logs", "fanout");
        // 创建临时队列
        String queue = channel.queueDeclare().getQueue();
        // 绑定交换机和队列 队列名称 交换机名称 路由
        channel.queueBind(queue, "logs", "");
        // 接收消息
        channel.basicConsume(queue, true, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("consumer1: " + new String(body, StandardCharsets.UTF_8));
            }
        });
    }
}

// consumer2
同consumer1，输出修改为：System.out.println("consumer2: " + new String(body, StandardCharsets.UTF_8));
```

**结果：**

```markdown
# consumer1 output...
consumer1: hello fanout, type [
consumer1: hello fanout

# consumer2 output...
consumer2: hello fanout, type [
consumer2: hello fanout
```

#### 4.4.4 Routing-路由模式

​		在发布订阅模式中，一条消息会被所有订阅的的队列消费。但是在某些场景下，我们希望不同的消息类型被不同的队列消费，这时就需要用到Routing模式的direct类型的Exchange。

```markdown
# 在Direct模式下：
	队列与交换机的绑定，不能是任意绑定了，而是需要指定一个Routing Keying，消息发送方在向Exchange发送消息时，也必须指定消息的Routing Key。
	Exchange也不再把消息交给每一个与之绑定的队列，而是根据消息的Routing Key判断，只有队列的Routing Key和消息的Routing Key完全一致时，才会发送消息。
```

![image](https://img-blog.csdnimg.cn/8f206f2cfdb34ce89c54e042a92cc21f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

```markdown
# 说明：
	P：Proudcer，生产者，向Exchange发送消息，同时会指定一个Routing Key。
	X：Exchange，交换机，接收生产者发送的消息，然后把消息传递给与routing key完全匹配的队列。
	C1：Consumer，消费者，其所在队列指定了需要routing key为error的消息
	C2：Consumer，消费者，其所在队列指定了需要routing key为info、error、warning的消息
```

**Producer：**

```java
public class RoutingProducer {

    public static void main(String[] args) throws IOException {
        // 获取连接
        Connection connection = RabbitMQUtil.getConnection();
        // 获取管道
        Channel channel = RabbitMQUtil.getChannel(connection);
        // 声明交换机
        channel.exchangeDeclare("logs_direct", "direct");
        // 发送消息
        String routingKey = "warning";
        channel.basicPublish("logs_direct", routingKey, null, String.format("routing for type [%s]", routingKey).getBytes(StandardCharsets.UTF_8));
        // 关闭资源
        RabbitMQUtil.close(channel, connection);
    }
}
```

**Consumer：**

```java
// consumer1
public class RoutingConsumer1 {

    public static void main(String[] args) throws IOException {
        Connection connection = RabbitMQUtil.getConnection();
        Channel channel = RabbitMQUtil.getChannel(connection);
        channel.exchangeDeclare("logs_direct", "direct");
        String queue = channel.queueDeclare().getQueue();
        // consumer1绑定routing key 为info
        channel.queueBind(queue, "logs_direct", "info");
        channel.basicConsume(queue, true, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("consumer1: " + new String(body, StandardCharsets.UTF_8));
            }
        });
    }
}

// consumer2
public class RoutingConsumer2 {

    public static void main(String[] args) throws IOException {
        Connection connection = RabbitMQUtil.getConnection();
        Channel channel = RabbitMQUtil.getChannel(connection);
        channel.exchangeDeclare("logs_direct", "direct");
        String queue = channel.queueDeclare().getQueue();
        // consumer1绑定routing key 为info、error、warning
        channel.queueBind(queue, "logs_direct", "info");
        channel.queueBind(queue, "logs_direct", "error");
        channel.queueBind(queue, "logs_direct", "warning");
        channel.basicConsume(queue, true, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("consumer2: " + new String(body, StandardCharsets.UTF_8));
            }
        });
    }
}
```

**结果：**

```markdown
# 依次发送routing key为info、error、warning结果：

## consummer1
consumer1: routing for type [info]

## consumer2
consumer1: routing for type [info]
consumer1: routing for type [error]
consumer1: routing for type [warning]
```

#### 4.4.5 Topics-主题（动态路由模式）

​		`Topics`模式同`Routing`模式相比，都可以根据Routing Key把消息路由到不同的队列。只不过`Topic`类型的Routing Key可以使用通配符。Routing Key一般是由一个或多个单词组成，多个单词之间以‘,’分割。Exchange的类型为`topic`。

![image](https://img-blog.csdnimg.cn/e1252695c8fd433e9885d1a9e8c4e9fb.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

```markdown
# 通配符：
	*（star）can substitute for exactly one word。精确匹配一个单词。
	#（hash）can substitute for zero or more words。匹配零个、一个或多个单词。
## 例如：
	audit.# ：匹配 audit、audit.irs、audit.irs.corporate 等。
	audit.* ：只能匹配 audit.irs。
```

**Producer：**

```java
public class TopicsProducer {

    public static void main(String[] args) throws IOException {
        Connection connection = RabbitMQUtil.getConnection();
        Channel channel = RabbitMQUtil.getChannel(connection);
        channel.exchangeDeclare("logs_topics", "topic");
        String routingKey = "user.save.now";
        channel.basicPublish("logs_topics", routingKey, null, String.format("hello topics, routing key : [%s]", routingKey).getBytes(StandardCharsets.UTF_8));
        RabbitMQUtil.close(channel, connection);
    }
}
```

**Consumer：**

```java
// consumer1
public class TopicsConsumer1 {

    public static final String EXCHANGE_NAME = "logs_topics";

    public static void main(String[] args) throws IOException {
        Connection connection = RabbitMQUtil.getConnection();
        Channel channel = RabbitMQUtil.getChannel(connection);
        channel.exchangeDeclare(EXCHANGE_NAME, "topic");
        String queue = channel.queueDeclare().getQueue();
        // 队列和交换机绑定的路由为 user.*
        channel.queueBind(queue, EXCHANGE_NAME, "user.*");
        channel.basicConsume(queue, false, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("consumer1: " + new String(body, StandardCharsets.UTF_8));
            }
        });
    }
}

// consumer2
public class TopicsConsumer2 {

    public static final String EXCHANGE_NAME = "logs_topics";

    public static void main(String[] args) throws IOException {
        Connection connection = RabbitMQUtil.getConnection();
        Channel channel = RabbitMQUtil.getChannel(connection);
        channel.exchangeDeclare(EXCHANGE_NAME, "topic");
        String queue = channel.queueDeclare().getQueue();
        // 队列和交换机绑定的路由为 user.*
        channel.queueBind(queue, EXCHANGE_NAME, "user.#");
        channel.basicConsume(queue, false, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("consumer2: " + new String(body, StandardCharsets.UTF_8));
            }
        });
    }
}
```

**结果：**

```markdown
# 依次发送routing key为user.save、user、user.save.now的结果：

## conumser1:
consumer1: hello topics, routing key : [user.save]

## consumer2:
consumer2: hello topics, routing key : [user.save]
consumer2: hello topics, routing key : [user]
consumer2: hello topics, routing key : [user.save.now]
```

## 5.SpringBoot重使用RabbitMQ

### 5.1 搭建环境

#### 5.1.1 引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

#### 5.1.2 配置信息

```yaml
spring:
  application:
    name: adu-rabbitmq
  rabbitmq:
    host: 10.2.3.4
    port: 5672
    username: adu
    password: adu
    virtual-host: /adu
```

### 5.2 各种模型客户端代码

#### 5.2.1 Hello World-直连模型

```java
/**
 * direct直连模式
 *
 * @author secret
 * @date 2021/12/14 6:30 PM
 */
@Component
@RabbitListener(queuesToDeclare = @Queue(value = "hello"))
public class HelloWorldConsumer {

    @RabbitHandler
    public void receive(String message) {
        System.out.println("consumer: " + message);
    }

}
```

#### 5.2.2 Work Queue - 工作队列模式

```java
/**
 * Work Queue模式-消费者
 * <p>
 * 循环模式，每个消费者得到的消息数相同
 *
 * @author secret
 * @date 2021/12/14 6:36 PM
 */
@Component
public class WorkQueueConsumer {

    @RabbitListener(queuesToDeclare = @Queue(value = "work", durable = "true", exclusive = "false", autoDelete = "false"))
    public void receive1(String message) {
        System.out.println("consumer1: " + message);
    }

    @RabbitListener(queuesToDeclare = @Queue(value = "work", durable = "true", exclusive = "false", autoDelete = "false"))
    public void receive2(String message) {
        System.out.println("consumer2: " + message);
    }

}
```

#### 5.2.3 Publish/Subscribe - 发布订阅模式

```java
/**
 * Publish/Subscribe模式
 * <p>
 * 使用交换机，将消息发给所有的绑定的队列
 *
 * @author secret
 * @date 2021/12/14 6:44 PM
 */
@Component
public class PublishSubscribeConsumer {

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue, // 没有参数的，表示临时队列
            exchange = @Exchange(value = "logs", type = "fanout") // 交换机
    ))
    public void receive1(String message) {
        System.out.println("consumer1: " + message);
    }

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue, // 没有参数的，表示临时队列
            exchange = @Exchange(value = "logs", type = "fanout") // 交换机
    ))
    public void receive2(String message) {
        System.out.println("consumer2: " + message);
    }
}
```

#### 5.2.4 Routing - 路由模式（direct）

```java
/**
 * Routing 模式 - 消费者
 * <p>
 * Exchange类型：direct
 *
 * @author secret
 * @date 2021/12/14 7:01 PM
 */
@Component
public class RoutingConsumer {

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue, // 临时队列
            exchange = @Exchange(value = "logs_direct", type = "direct"),
            key = "info" // routing key
    ))
    public void receive1(String message) {
        System.out.println("consumer1: " + message);
    }


    @RabbitListener(bindings = @QueueBinding(
            value = @Queue, // 临时队列
            exchange = @Exchange(value = "logs_direct", type = "direct"),
            key = {"info", "error", "warning"} // routing key
    ))
    public void receive2(String message) {
        System.out.println("consumer2: " + message);
    }

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue, // 临时队列
            exchange = @Exchange(value = "logs_direct", type = "direct"),
            key = {"warning"}
    ))
    public void receive3(String message) {
        System.out.println("consumer3: " + message);
    }
}
```

#### 5.2.5 Topics - 主题（动态路由）模式

```java
/**
 * Topics模式 -消费者
 *
 * Exchange的类型为：topic
 *
 * @author secret
 * @date 2021/12/14 7:16 PM
 */
@Component
public class TopicsConsumer {

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue, // 临时队列
            exchange = @Exchange(value = "logs_topics", type = "topic"),
            key = {"user.*"}
    ))
    public void receive1(String message) {
        System.out.println("consumer1: " + message);
    }

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue, // 临时队列
            exchange = @Exchange(value = "logs_topics", type = "topic"),
            key = {"user.#"}
    ))
    public void receive2(String message) {
        System.out.println("consumer2: " + message);
    }

}
```

#### 5.2.6 客户端测试代码

```java
/**
 * 生产端没有指定交换机，只有routing key和object。通过生产端的routing key和消费端的队列匹配
 */
@SpringBootTest(classes = RabbitMQApplication.class, webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@RunWith(SpringRunner.class)
@EnableAutoConfiguration
public class RabbitMQApplicationTest {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    // 直连模式
    @Test
    public void testHelloWorld() {
        rabbitTemplate.convertAndSend("hello", "hello spring boot starter amqp...");
    }
	// Work Queue模式
    @Test
    public void testWorkQueue() {
        for (int i = 0; i < 20; i++) {
            rabbitTemplate.convertAndSend("work", "hello work queue" + i);
        }
    }
	// Publish/Subscribe模式
    @Test
    public void testPublishSubscribe() {
        for (int i = 0; i < 10; i++) {
            rabbitTemplate.convertAndSend("logs", "", "hello publish subscribe " + i);
        }
    }
	// Routing模式
    @Test
    public void testRouting() {
        String routingKey = "warning";
        rabbitTemplate.convertAndSend("logs_direct", routingKey, "hello routing, type=" + routingKey);
    }
	// Topics模式
    @Test
    public void testTopics() {
        String routingKey = "user.save";
        rabbitTemplate.convertAndSend("logs_topics", routingKey, "hello topics, type=" + routingKey);
    }
}
```

## 6.MQ的应用场景

### 6.1 异步处理

```markdown
# 场景说明：
	用户注册后，需要发送邮件和注册短信，传统的做法有2种：1）串行方式 2）并行方式
## 串行方式
	将注册信息写入数据库后，再发送邮件，再注册短信，以上三个任务全部返回结果后，才返回给客户端。这里有一个问题，邮件、短信并不是必须的，它只是一个通知，而这种做法需要让客户端等待没有必要等待的东西。
## 并行方式
	将注册信息写入数据库后，发送邮件的同时，发送短信。以上三个任务完成后，返回给客户端，并行的方式能提高处理的效率。
```

![image](https://img-blog.csdnimg.cn/c0e203b0f649433984cbd50434586974.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

![image](https://img-blog.csdnimg.cn/17a5dbf434ee4c76a2745392fb70a774.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

```markdown
# 消息队列：
	除了以上提到的2种传统方式之外，我们还可以选择 消息队列 的形式。因为发送邮件和发送短信不是必须的，所以我们可以使用消息队列的形式，进行异步处理，相对于并行方式，减少了发送邮件或者短信的时间，相当于只有 注册信息写入数据库 + 写入消息队列 这个操作的耗时。
```

![image](https://img-blog.csdnimg.cn/c2bad0734fff4638acb0f0e7aa222327.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

### 6.2 应用解藕

![image](https://img-blog.csdnimg.cn/4096d6dc33ff48e380b3ec742e513b3c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

```markdown
# 场景：
	双十一是购物节，用户下单后，订单系统需要通知库存系统，传统做法是订单系统调用库存系统提供的接口。
# 缺点：
	这种做法有个缺点：当库存系统异常时，订单系统就无法使用。订单系统和库存系统就出现了耦合。解决方法引入消息队列。
```

![image](https://img-blog.csdnimg.cn/129b02a2439545ed8f748471c0f21b8a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

```markdown
# 引入消息队列：
	订单系统：用户下单后，订单系统完成持久化后，将消息写入 消息队列 ，返回用户下单成功。
	库存系统：订阅下单消息，获取下单消息，进行库存操作。就算库存系统出现故障，消息队列也能保证消息的可靠投递，不会导致消息丢失。
```

### 6.3 流量削峰

```markdown
# 场景：
	秒杀活动，一般会因为流量过大，导致应用挂掉。为了解决这个问题，一般在应用之前加入消息队列。
# 作用：
	1.可以控制活动人数，超过一定阈值的订单直接丢弃
	2.可以缓解短时间的高流量压垮应用（程序按自己处理能力获取订单）。
# 流程：
	1.用户的请求，服务器收到之后先加入到消息队列。假如超过消息队列的最大长度，则直接丢弃用户请求或者跳转到错误页面。
	2.秒杀业务根据消息队列中的请求信息，再做后续的处理。
```

## 7.RabbitMQ集群

### 7.1 集群架构（副本集群）

> 默认情况下：RabbitMQ代理操作所需的所有数据/状态都将跨所有节点复制。这方面的一个例外是 消息队列，默认情况下，消息队列位于一个节点上，尽管他们可以从任意节点上看到和访问。--官网
>
> 也就是说，集群架构的方式只能在节点之间同步（复制）交换机等基本信息，无法同步队列信息。而队列信息是保存数据的地方，如果无法在节点之间进行复制，那么集群将失去一些意义。

#### 7.1.1 架构图

![image](https://img-blog.csdnimg.cn/1b08e25a4092495f8412a4dd0801b780.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

```markdown
# 核心解决问题：
	当集群中某一时刻Master节点宕机，可以对Queue中信息进行备份。
```

#### 7.1.2 集群搭建

```markdown
PS：使用docker安装
# 0.集群规划
	node1:  172.18.0.2 5672 15672 		master 主节点
	node2:	172.18.0.3 5673 15673		repl1  副本节点
	node3:	172.18.0.4 5674 15674	 	repl2  副本节点
# 1.克隆三台主机名和ip映射，并修改各个虚拟机的hostname
	创建网络：
		docker network create rabbit-net
	创建容器：
		docker run -d --privileged=true --hostname node1 --name node1 -p 15672:15672 -p 5672:5672 --network rabbit-net centos:8.2.2004 /sbin/init
	进入容器：
		docker exec -it node/node1/node2 /bin/bash
	复制erlang、rabbit的rpm包到容器：
		docker cp ./erlang.xxx.rpm node1:/
		docker cp ./rabbitmq.xxx.rpm node1:/
	安装：
		rpm -ihv erlang.xxx.rpm
		rpm -ivh rabbitmq.xxx.rpm --nodeps --force
	配置：
		在/etc/hosts加入各个节点的ip和域名对应关系。
	问题：
		参考：https://blog.csdn.net/weixin_42181917/article/details/105579288
		1.System has not been booted with systemd as init system (PID 1). Can't operate.Failed to create bus connection: Host is down
		解决：创建容器使用 /sbin/init
		2.Could not set property: Failed to set static hostname: Device or resource busy
		解决：退出容器，重新进入在设置一次
			hostnamectl set-hostname node/node1/node2
			exit
			hostnamectl set-hostname node/node1/node2
# 2.三台机器安装rabbitmq，并同步cookie文件
	docker cp 命令将宿主机中的erlang和rabbitmq的rpm包都上传到虚拟机中，启动rabbitmq。
	docker cp 命令将node1节点中的 /var/lib/rabbitmq/.erlang.cookie文件复制到宿主机，再通过宿主机复制到node1和node2.
	这里需要注意看是否.erlang.cookie文件的权限是否和node相同：chown、chgrp 命令可以修改权限。
# 3.查看cookie文件是否一致
	进入三个节点的虚拟机中，执行: cat /var/lib/rabbitmq/.erlang.cookie，查看内容是否一致。
# 4.后台启动rabbit
	rabbitmq-server -detached
# 5.在node2和node3执行加入集群命令
	1.关闭		rabbitmqctl stop_app
	2.加入集群	   rabbitmqctl join_cluster rabbit@node
	3.启动服务	   rabbitmqctl start_app
	

```

**搭建成功：**

![image](https://img-blog.csdnimg.cn/4e5b989ee58b4a63816087ee87cb34d4.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

![image](https://img-blog.csdnimg.cn/c9e21a1431b74730908aefb32fdc5cbe.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

### 7.2 镜像集群

> 镜像队列机制就是将队列在三个节点之间设置主从关系，消息会在三个节点之间自动进行同步。且如果其中一个节点不可用，并不会导致消息丢失或者服务不可用的情况，提升MQ集群整体的高可用性。--摘自官网

#### 7.2.1 架构图

![image](https://img-blog.csdnimg.cn/677182699180463baa2a64e582a7042c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

#### 7.2.2 集群搭建

​		镜像集群的搭建，是在`副本集群`的基础之上做的额外配置，也就是说必选先搭建好`副本集群`才能搭建镜像集群。这个额外配置就是需要创建一个`策略`。

```markdown
# 0.策略说明
	rabbitmqctl set_policy [--vhost <vhost>] [--priority <priority>] [--apply-to <apply-to>] <name> <pattern> <definition>
	--vhost：可选参数，针对指定virutal host下的queue进行设置
	--priority：可选参数，policy的优先级，越大越优先
	--appliy-to：可选参数，表示策略应用的对象，后面跟queues|exchanges|all。
	name：策略名称，唯一标识。
	pattern：匹配队列的正则表达式。
	definition：镜像定义，包括三个部分：ha-mode、ha-params、ha-sync-mode
		ha-mode：镜像队列模式，可选值：all/exaclty/nodes
			all：表示在集群的所有节点上进行镜像
			exactly：表示在指定个数的节点上进行镜像，节点个数通过ha-params指定
			nodes：表示在指定节点上进行镜像，节点名称通过ha-params指定
		ha-params：ha-mode需要用到的参数
		ha-sync-mode：队列同步方式，可选值为：automatic/manual
			automatic：自动同步
			manual：用户触发同步
# 1.查看当前策略
	rabbitmqctl list_policies
# 2.添加策略
	rabbitmqctl set_policy ha-all '^hello' '{"ha-mode":"all","ha-sync-mode":"automatic"}'
	说明：策略正则表示为"^"表示匹配所有队列，"^hello"表示匹配hello开头的队列
# 3.删除策略
	rabbitmqctl clear_policy ha-all
```

**搭建成功：**

![image](https://img-blog.csdnimg.cn/0a123778671941e99aac309f1180c599.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

![image](https://img-blog.csdnimg.cn/4c70637a590c4150b0f2296b65bc3837.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)
