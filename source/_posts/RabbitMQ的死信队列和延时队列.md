---
title: RabbitMQ的死信队列和延时队列
top: false
date: 2021-12-26 14:14:49
tags:
    - 中间件
    - RabbitMQ
categories:
    - 中间件
---

# RabbitMQ死信队列和延时队列

​		RabbitMQ本身是具有`死信队列`和`死信交换机`属性的，`延时队列` 可以通过死信队列和死信交换机来实现。在电商行业中，通常都会有一个需求：订单超时未支付，自动取消该订单。那么通过RabbitMQ实现的延时队列就是实现该需求的一种方式。

## 1、死信队列

​		`死信`顾名思义，就是死掉的信息，英文是Dead Letter。`死信交换机（Dead-Letter-Exchange）`和普通交换机没有区别，都是可以接受信息并转发到与之绑定并能路由到的队列，区别在于`死信交换机`是转发`死信`的，而和该`死信交换机`绑定的队列就是`死信队列`。说的再通俗一点，死信交换机和死信队列其实都只是普通的交换机和队列，只不过接受、转发的信息是`死信`，其他操作并没有区别。

### 1.1 死信的条件

​		称为`死信`的信息，需要如下几个条件：

- 消息被消费者拒绝（通过basic.reject 或者 back.nack），并且设置 requeue=false。
- 消息过期，因为队列设置了TTL（Time To Live）时间。
- 消息被丢弃，因为超过了队列的长度限制。

这时以上几个条件的方式基本上都有2种：1）`rabbitmqctl`命令行设置policy（策略）参数； 2）硬编码，也就是在代码中设置。

### 1.2 消费者拒绝

#### 1.2.1 编码方式

​		硬编码就是在代码中编写**业务队列声明时**对应的参数：

- `x-dead-letter-exchange`：死信交换机，必须
- `x-dead-letter-routing-key`：死信交换机转发到死信队列的路由键，可选

**Producer:**

```java
// producer
public class RejectProducer {
	// 定义业务交换机
    public static final String ORDER_X = "order.exchange";
	// main方法
    public static void main(String[] args) throws IOException {
        // 获取连接
        Connection connection = RabbitMQUtil.getConnection();
        // 获取通道
        Channel channel = RabbitMQUtil.getChannel(connection);
        // 发消息，路由键分别为：d.order.123、d.other.123、d
        // 1）d.order.123：业务consumer会收到消息，order死信队列也会收到消息
        // 2）d.other.123：业务consumer会收到消息，other死信队列也会收到消息
        // 3）d：只有业务consumer会收到消息
        channel.basicPublish(ORDER_X, "d.order.123", null, "hello my friend".getBytes(StandardCharsets.UTF_8));
        // 关闭资源
        RabbitMQUtil.close(channel, connection);
    }
}
```

**Consumer:**

```java
/**
 * 死信队列条件：消费者拒绝
 * 1）basic.reject(tag, requeue) 表示拒绝接受消息，第二个参数表示是否重新入队，如果为true，可能造成死循环，需要注意
 * 2）basic.nack(tag, multi, requeue) multi可以同时拒绝多条，requeue和reject相同。nack表示未ack的数据，会有单独的标识
 */
public class RejectConsumer {

    public static final String DEAD_LETTER_X = "dead.letter.exchange";
    public static final String DEAD_LETTER_Q_1 = "dead.letter.queue.order";
    public static final String DEAD_LETTER_Q_2 = "dead.letter.queue.other";
    public static final String ORDER_X = "order.exchange";
    public static final String ORDER_Q = "order.queue";

    public static void main(String[] args) throws IOException {
        // reject和nack方式
        rejectAndNack();
    }

    public static void rejectAndNack() throws IOException {
        // 获取连接
        Connection connection = RabbitMQUtil.getConnection();
        // 获取通道
        Channel channel = RabbitMQUtil.getChannel(connection);
        // 声明死信队列和死信交换机
        declareOrderDLX(channel);
        declareOtherDLX(channel);
        
        // 声明业务交换机
        channel.exchangeDeclare(ORDER_X, "topic", false, true, null);
        // 设置业务参数
        Map<String, Object> arguments = new HashMap<>();
        // 设置死信交换机，一个死信交换机绑定了两个死信队列，根据路由键来区分消息的转发
        arguments.put("x-dead-letter-exchange", DEAD_LETTER_X);
        // 设置死信队列路由key：
        // 如果这里设置了路由键，则publish过来的消息再转发到死信交换机时，以该路由键转发到死信队列，
        // 如果未设置该参数，则按照publish时的路由键转发到死信队列
//        arguments.put("x-dead-letter-routing-key", "d.other");
        // 声明业务队列
        channel.queueDeclare(ORDER_Q, false, false, true, arguments);
        // 绑定业务队列和业务交换机
        channel.queueBind(ORDER_Q, ORDER_X, "#");
        // 监听业务队列消息
        channel.basicConsume(ORDER_Q, false, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("order consumer: " + new String(body, StandardCharsets.UTF_8));
                // 拒绝
                channel.basicReject(envelope.getDeliveryTag(), false);
                System.out.println("order properties: " + envelope.toString());
            }
        });
    }

    // 声明order死信队列
    public static void declareOrderDLX(Channel channel) throws IOException {
        // 声明交换机，只是为了模拟，设置的duriable=false，autodelete=true
        channel.exchangeDeclare(DEAD_LETTER_X, "topic", false, true, null);
        // 声明队列
        channel.queueDeclare(DEAD_LETTER_Q_1, false, false, true, null);
        // 绑定交换机和队列，只接受 *.order.# 规则的消息
        channel.queueBind(DEAD_LETTER_Q_1, DEAD_LETTER_X, "*.order.#");
        // 监听消息
        channel.basicConsume(DEAD_LETTER_Q_1, true, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("dead letter consumer 【order】: " + new String(body, StandardCharsets.UTF_8));
                System.out.println("dead letter properties 【order】:" + envelope.toString());
            }
        });
    }

    // 声明other死信队列
    public static void declareOtherDLX(Channel channel) throws IOException {
        // 声明交换机
        channel.exchangeDeclare(DEAD_LETTER_X, "topic", false, true, null);
        // 声明队列
        channel.queueDeclare(DEAD_LETTER_Q_2, false, false, true, null);
        // 绑定交换机和队列，只接受 *.other.# 规则的消息
        channel.queueBind(DEAD_LETTER_Q_2, DEAD_LETTER_X, "*.other.#");
        // 监听消息
        channel.basicConsume(DEAD_LETTER_Q_2, true, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("dead letter consumer 【other】: " + new String(body, StandardCharsets.UTF_8));
                System.out.println("dead letter properties 【other】:" + envelope.toString());
            }
        });
    }
}
```

**总结：**

**1）** consumer方使用`basic.reject`或者`basic.nack`都会将消息转发到匹配的死信队列（requeue=false），区别在于`basic.reject`相比`basic.nack`少一个参数`mutil`，表示是否批量back。而且nack的数量可以再web端看到。

**2）** 使用`x-letter-dead-exchange`设置死信交换机，这个是必须设置的。`x-letter-dead-routing-key`设置死信交换机转发到死信队列的路由键，相当于重新定义了publish的路由键，该参数可选，可以根据具体业务判断是否需要设置。

#### 1.1.2 策略方式

策略方式需要在rabbitmq的服务器上执行如下命令：

```bash
rabbitmqctl set_policy {策略名称} ".*" '{"dead-letter-exchange":"my-dlx"}' --apply-to queues
```

例如：

```bash
rabbitmqctl set_policy dlx "dead.*" '{"dead-letter-exchange":"test-dead-letter-exchange"}' --apply-to queues
```

表示给所有的 *以dead.开头* 的队列设置死信交换机 *test-dead-letter-exchange* ，策略名字为 *dlx*。然后我们在rabbitmq的web界面新建一个名字为`test-dead-letter-exchange`的exchange，并且新建名为`dead.order.queue`和`dead.other.queue`的queue，绑定`test-dead-letter-exchange`，路由键分别为：`order.#`和`other.#`。

![死信交换机绑定死信队列](https://img-blog.csdnimg.cn/9b1d1dd66aeb4e7292f0058df0a368b6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

> 说明：由于我们要模拟的死信转发到死信队列的情况，所以这两个新建的queue都设置了ttl为10000ms，也就是10s。

![死信队列](https://img-blog.csdnimg.cn/dbdbef64985544f8afdd75dc4cb06712.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)		

我们看到消息在10s之后成功到了`dead.order.queue`，说明我们的配置生效。这里我将这个过程画个图：

![绑定死信队列](https://img-blog.csdnimg.cn/d6d9c46a56fc4a0cb9722c5000360189.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

### 1.3 设置过期时间

> 文档：https://www.rabbitmq.com/ttl.html

​		我们可以给 *队列* 或者 *消息* 设置过期时间。*队列* 的过期时间，类似于 `autoDelete` 参数，表示队列在指定时长内如果没有使用的话会被删除，队列没有使用者，队列最近未重新声明（重新声明续订租约），以及`basic.get`至少在过期期间未被调用，例如，这可以用于RPC样式的回复队列，其中可以创建许多可能永远不会被耗尽的队列。*消息* 的过期时间我们可以在发消息时设置在消息体，也可以给这个队列设置一个消息过期时间，其实就是两种方式，一种设置在队列上，另一种是设置在消息上。

#### 1.3.1 编码方式

**1）设置消息体过期时间**

```java
// 设置消息属性
AMQP.BasicProperties properties = new AMQP.BasicProperties.Builder()
    .expiration("2000") // 设置消息过期时间，2s，既是所在队列没有过期时间也可以
    .build();
// 发消息
channel.basicPublish("hello", "order.123", properties, "hello my friend".getBytes(StandardCharsets.UTF_8));
```

**2）设置队列的消息过期时间**

```java
// 设置参数
Map<String, Object> arguments = new HashMap<>();
// 设置队列的过期时间，5s
arguments.put("x-message-ttl", 5000);
// 设置死信队列
arguments.put("x-dead-letter-exchange", DEAD_LETTER_X);
// 声明队列
channel.queueDeclare(ORDER_Q, false, false, true, arguments);
```

我们可以看到，创建好队列之后会有个`TTL`标识，`x-message-ttl`标识该队列设置的消息过期时间为5s。

![队列的消息过期时间](https://img-blog.csdnimg.cn/c05ac2bbdb3049db94b434e4e3c508d6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

**3）设置队列的过期时间**

```java
// 设置队列参数
Map<String, Object> arguments = new HashMap<>();
// 设置队列中消息的过期时间，5s
arguments.put("x-message-ttl", 5000);
// 设置队列的过期时间，10s如果队列未使用（未操作），则删除队列
arguments.put("x-expires", 10000);
// 声明队列
channel.queueDeclare(ORDER_Q, true, false, false, arguments);
```

**注意：无论队列中是否存在消息，如果没有操作队列，就会被自动删除。**

#### 1.3.2 策略方式

**1）设置队列的消息过期时间**

```bash
rabbitmqctl set_policy --vhost /adu TTL ".*" '{"message-ttl":60000}' --apply-to queues
```

表示在 `/adu`虚拟主机下增加一个名称为 *TTL* 策略，设所有队列 `message-ttl`消息过期时间60s。

**2）设置队列的过期时间**

```bash
rabbitmqctl set_policy --vhost /adu expiry ".*" '{"expires":1800000}' --apply-to queues
```

表示在`/adu`虚拟主机下增加一个名称为 *expiry* 的策略，设置所有的队列过期时间为180s。

### 1.4 超过队列长度

​		默认情况下，队列没有长度限制（但是总归是有硬盘和内存的限制的）。我们可以显示的设置队列的长度，可以是消息的**数量限制**，也可以是队列总消息内容的**占用内存长度**，或者两种都设置。一个队列的最大长度，可以使用 *策略* 或者 *编码* 的方式进行设置，或者在创建队列时web界面设置。如果 `策略` 方式和 `编码` 方式都设置了，则 `值更小的` 会生效。

​		如果队列设置了队列长度限制，那么当队列中的消息达到最大长度时，默认的 *溢出* 规则为 *丢弃最老的消息（队列头部）*。我们可以改变这个规则，使用 `overflow` 参数来配置。`overflow` 可选值为 `x-reject-publish`或者`x-reject-publish-dls` ，两者都表示拒绝接受新消息，区别在于 `reject-publish-dlx` 也会导致死信拒绝消息<sup>1</sup>。

> 此处有个疑问：reject-publish-dlx和reject-publish的区别问题。针对官网的翻译，我觉得 `reject-publish-dlx` 是与之绑定的死信队列不会收到消息， `reject-publish` 相反会收到消息。但是做实验的时候刚好和我理解的相反？熟悉的铁子们回复说一下哈呀。

#### 1.4.1 编码方式

​		使用`x-max-length`和`x-max-length-bytes`参数设置。

```java
// 设置参数
Map<String, Object> arguments = new HashMap<>();
// 设置队列最大长度，5条消息
arguments.put("x-max-length", 5);
// 队列溢出的策略：drop-head（默认）、reject-publish、reject-publish-dlx
//        arguments.put("x-overflow", "reject-publish-dlx");
arguments.put("x-overflow", "reject-publish");
// 设置死信队列
arguments.put("x-dead-letter-exchange", DEAD_LETTER_X);
// 声明队列
channel.queueDeclare(ORDER_Q, false, false, true, arguments);
```

#### 1.4.2 策略方式

```bash
rabbitmqctl set_policy --vhost /adu limit "^five_msg" '{"dead-letter-exchange":"test-dead-letter-exchange","max-length":5,"overflow":"reject-publish-dlx"}' --apply-to queues
```

表示在 `/adu`虚拟主机下增加一个名称为 *limit* 策略，设所有以 *five_msg* 开头的队列 *消息最多为5个*、*消息溢出策略为拒绝*、*设置死信交换机* 。

> 设置 `max-length-bytes` 也是同样的方式。

## 2、延时队列

​		延时队列，顾名思义就是存放延时消息的队列，也就是说消费者在一定的延时后才会收到消息。典型的应用场景就是如上所述的订单超时未支付自动取消。

### 2.1 借助死信队列实现

​		其实在介绍完 *死信队列* 之后，就能大概看出来如何使用 *死信队列* 来实现延时队列了。就是使用消息的`TTL` 属性，将过期的消息转发到死信队列中，业务监听死信队列的消息就行了。这种情况适合给队列设置消息过期时间的情况，就是队列中所有的消息都是同一个过期时间，到期按照顺序转发到死信队列中，不会有问题。

​		如果消息的过期时间是在发消息的时候设置在消息体上的，可能会出问题。比如按顺序发送msg1和msg2两条消息，msg1的过期时间为5s，msg2的过期时间为2s。正常理解下，结果肯定是msg2先到死信队列被消费，但是结果却是两条消息都在5s时转发到死信队列被消费。其实比较好理解，因为队列的特性就是 *先进先出* ，即使msg2先到了过期时间，但是msg1在它之前阻塞，只有msg1被消费了，msg2才能到队头被消费。 我们画个图：

![死信队列实现延时队列问题](https://img-blog.csdnimg.cn/15894c373b12405381d8e89b2383a82b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

### 2.2 借助RabbitMQ插件实现

​		rabbitmq提供了一个插件 `rabbitmq_delayed_message_exchange` 让我们能够实现 *延迟队列* 的效果，同时能够解决 *通过死信队列实现延迟队列* 出现的消息阻塞问题。该插件从RabbitMQ的3.6.12开始支持，要确认当前自己的rabbitmq版本是否支持该插件。

#### 2.2.1 下载插件

> 下载地址：https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases

​		下载该插件后，将 `rabbitmq_delayed_message_exchange-3.9.0.ez` 包放到 *RabbitMQ安装目录的plugins* 目录下：

![插件位置](https://img-blog.csdnimg.cn/bb1b98bbafdf455dbd9a38491aeefccd.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

#### 2.2.2 启用插件

执行控制台命令，重启rabbitmq服务：

```bash
# 1.列出所有插件
rabbitmq-plugins list
# 2.启用rabbitmq_delayed_message_exchange
rabbitmq-plubins enable rabbitmq_delayed_message_exchange
# 3.重启服务（好像可以不用重启）
systemctl restart rabbitmq-server.service
```

​		在此之后，web界面的 *exchanges* 便可以创建type为 `x-delayed-message` 的交换机，或者在代码中声明该类型的交换机。要是用其延时功能，需要在发消息的时候加一个 header ：`x-delay=xxx` ，表示延时xxx毫秒。

```java
// 声明延时交换机，type=x-delayed-message，x-delayed-type=direct|fanout|topic
Map<String, Object> args = new HashMap<>();
args.put("x-delayed-type", "topic"); // 相当于之前exchange的type
channel.exchangeDeclare(EXCHANGE_NAME, "x-delayed-message", false, true, args);

// 发送消息，x-delay，值为过期时间
Map<String, Object> headers = new HashMap<>();
headers.put("x-delay", 5000); // 5s
AMQP.BasicProperties props = new AMQP.BasicProperties().builder()
    .headers(headers)
    .build();
// 发送消息
channel.basicPublish(EXCHANGE_NAME, "other.save", props, "i am 5s message".getBytes(StandardCharsets.UTF_8));
```

![添加延时交换机](https://img-blog.csdnimg.cn/106078170371495d9f19780059e92d51.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

## 3、总结

​		上面我们提到了使用RabbitMQ实现延时队列功的方案：1）借助本事的死信队列实现，监听死信队列；2）借助插件实现。优缺点如下：

- 死信队列实现方式，需要在队列上设置消息过期时间，不灵活；需要再多用一个死信队列，占用空间；rabbitmq本事自带死信队列，实现方便。
- 插件实现方式，需要下载安装插件，要考虑版本兼容性；代码逻辑简单，容易上手。

​		回到我们开头的需求：订单支付超时自动取消。这个功能主要就是需要一个延时队列，那通过rabbitmq实现延时队列只是一种方式，还可以通过其他方式实现，比如Java的 `DelayQueue` 、`Quartz定时任务`、`Redis的zset`、`时间轮` 等都可以实现，具体方案还是要结合项目和具体方式的优缺点来选择。比如项目中使用到了RabbitMQ，那使用RabbitMQ实现延迟队列就是比较好的方式，那具体选择插件方式还是死信队列方式，还需要看项目中对该功能的灵活程度来选择。

**参考：**

1. [https://www.rabbitmq.com/dlx.html](https://www.rabbitmq.com/dlx.html)
2. [https://www.cnblogs.com/williamwsj/p/8108970.html](https://www.cnblogs.com/williamwsj/p/8108970.html)
3. [https://www.jianshu.com/p/256d2eaf1786](https://www.jianshu.com/p/256d2eaf1786)
4. [https://www.rabbitmq.com/community-plugins.html](https://www.rabbitmq.com/community-plugins.html)
5. [https://blog.csdn.net/zhenghongcs/article/details/106700446](https://blog.csdn.net/zhenghongcs/article/details/106700446)

------

最后：因小的才疏学浅，如有问题，请不吝指出，感谢感谢～
