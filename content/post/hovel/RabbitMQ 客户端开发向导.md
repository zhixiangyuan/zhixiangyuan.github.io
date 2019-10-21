---
title: "RabbitMQ 客户端开发向导"
date: 2019-10-21T11:12:48+08:00
keywords: []
description: ""
tags: [
    "RabbitMQ"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
draft: true
---

# 1 引入依赖

```xml
        <dependency>
            <groupId>com.rabbitmq</groupId>
            <artifactId>amqp-client</artifactId>
            <version>5.4.3</version>
        </dependency>
```

# 2 连接 RabbitMQ 

## 2.1 通过给定参数来连接

```java
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUsername("<username>");
        factory.setPassword("<password>");
        factory.setVirtualHost("<virtual_host");
        factory.setHost("<host>");
        factory.setPort(5672);
        Connection connection = factory.newConnection();
```

## 2.2 通过给定 URI 的方式来连接

```java
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUri("amqp://<userName>:<password>@<ipAddress>:<port>/<virtualHos>");
        Connection connection = factory.newConnection();
        // 不过这里需要注意，当 <virtualHos> 是 '/' 时，需要写成 %2F
        // 否则连接时会报 vhost virtualHost not found
```

## 2.3 Connection 使用注意事项

1. `Channel channel = connection.createChannel();`，通过 connection 可以创建一个 channel，但是 channel 是线程不安全的，不能多线程共享使用。
2. Connection 中有一个 isOpen 方法 它的内部实现如下所示，所以不能多线程去判断 isOpen，否则就会出现锁的竞争降低性能。一般在创建完 Connection 之后默认连接成功，如果连接失败在使用的时候会报出 ShutdownSignalException 异常，我们只需要捕获这个异常即可。

```java
    // Connection isOpen 的内部实现
    public boolean isOpen() {
        synchronized(this.monitor) {
            return this.shutdownCause == null;
        }
    }
```

## 2.4 Channel 使用注意事项

Channel 中同样有一个 isOpen 方法，我们也不要使用它，创建完 Channel 之后默认能够使用，如果关闭了则会报出 ShutdownSignalException 异常，我们捕获这个异常即可。

# 3 交换器和队列相关方法

使用交换器与队列之前需要先声明，如果声明的一个已存在的交换器，如果交换器参数完全匹配则什么都不做，如果参数不匹配则会抛出异常。

## 3.1 声明交换机的相关方法

```java
    AMQP.Exchange.DeclareOk exchangeDeclare(
            // 交换机的名称
            String exchange,
            // type 有四种 direct、fanout、topic、headers
            // jar 包里面有一个 BuiltinExchangeType 定义了这四种
            // 类型，可以直接取用
            String type,
            // durable 是否设置为持久化，持久化可以将交换机存盘
            // 在服务器重启的时候不会丢失相关信息
            boolean durable,
            // 是否自动删除，前提条件是至少有一个队列或者交换机
            // 与这个交换机绑定，此后所有队列和交换机都解绑，那么
            // 这个交换机也会自动删除
            boolean autoDelete,
            // 是否设置为内置交换机，如果设置为内置交换机，那么客户
            // 端程序无法直接发送消息到这个交换机中，只能通过交换机
            // 路由到交换机这种方式
            boolean internal,
            // 其他的一些参数
            Map<String, Object> arguments
    ) throws IOException;

    // 还有类似的方法
    // 区别有两个，1. 返回值变成 void，2. type 变成 BuiltinExchangeType 类型
    void exchangeDeclareNoWait(String exchange,
        BuiltinExchangeType type,
        boolean durable,
        boolean autoDelete,
        boolean internal,
        Map<String, Object> arguments
    ) throws IOException;
```

exchangeDeclareNoWait 比 exchangeDeclare 多设置了一个 nowait 参数，这个 nowait 参数指的是 AMQP 中 Exchange.Declare 命令的参数，意思是不需要服务端返回，而普通的 exchangeDeclare 在声明一个交换机之后，需要等待服务器返回 Exchange.Declare-Ok 这个 AMQP 命令。

不过如果服务器还未创建交换机则使用交换机必然发生异常，所以不建议使用 exchangeDeclareNoWait 这个方法。

```java
    // 这个方法会判断交换机是否存在，如果存在则正常返回，如果不存在
    // 则抛出异常同时 Channel 也会被关闭。
    Exchange.DeclareOk exchangeDeclarePassive(String name) throws IOException;
```

## 3.2 删除交换机的相关方法

```java
    // 无论当前交换机是否使用都直接删除
    Exchange.DeleteOk exchangeDelete(String exchange) throws IOException;

    // ifUnused 为 true 则只在交换机未使用的情况下删除该交换机
    // ifUnused 为 false 则不论交换机是否使用都删除该交换机
    Exchange.DeleteOk exchangeDelete(String exchange, boolean ifUnused) throws IOException;

    // 设置 nowait 参数，不需要服务器响应
    void exchangeDeleteNoWait(String exchange, boolean ifUnused) throws IOException;
```

## 3.3 声明队列的相关方法

queueDeclare 有两个重载方法

```java
    // 不带任何参数的 queueDeclare 方法默认创建一个由 RabbitMQ 命令
    // 的（类似 amq.gen-LhQzlgv3GhDOv8PIDabOXA 名称，这种队列也称之
    // 为匿名队列）、排他的、自动删除的、非持久化的队列。
    Queue.DeclareOk queueDeclare() throws IOException;

    AMQP.Queue.DeclareOk queueDeclare(
            // 队列名称
            String queue, 
            // 是否持久化，持久化的队列会存盘，在服务器重启的时候可
            // 以保证不丢失相关信息。
            boolean durable, 
            // 是否排他，如果一个队列被声明为排他队列，该队列仅对首
            // 次声明它的连接可见，并在连接断开时自动删除。这里需要
            // 注意三点：
            // 1. 排他队列是基于连接（Connection）可见的，同一个连接
            // 的不同通道（Channel）是可以同时访问同一连接创建的排他
            // 队列的。
            // 2. 首次是指如果一个连接已经声明了一个排他队列，其他连
            // 接是不允许建立同名的排他队列的，这个与普通队列不同
            // 3. 即使该队列是持久化的，一旦连接关闭或客户端退出，该
            // 排他队列都会被自动删除，这种队列适用于一个客户端同时发
            // 送和读取消息的应用场景。
            boolean exclusive,
            // 是否自动删除，前提是至少有一个消费者连接到这个队列，之后
            // 所有与这个队列连接的消费者都断开才会自动删除。
            boolean autoDelete,
            // 设置队列的一些参数
            Map<String, Object> arguments
    ) throws IOException;
```

生产者和消费者都能够声明队列，但是如果消费者在同一个信道上订阅了另一个队列，就无法再声明队列了，必须先取消订阅，然后将新到置为“传输”模式，之后才能声明队列。

## 3.4 删除队列的相关方法

```java
    // 删除队列，无论队列是否在使用
    Queue.DeleteOk queueDelete(String queue) throws IOException;

    // ifUnused 为 true 仅在未使用时删除，true 不论是否在使用
    // ifEmpty 当队列中没有消息堆积时才删除
    Queue.DeleteOk queueDelete(String queue, boolean ifUnused, boolean ifEmpty) throws IOException;

    // 同上，设置 nowait 参数，不需要服务器返回
    void queueDeleteNoWait(String queue, boolean ifUnused, boolean ifEmpty) throws IOException;
```

## 3.5 清空队列中的数据

```java
    // 清空队列中的数据
    Queue.PurgeOk queuePurge(String queue) throws IOException;
```

## 3.6 队列绑定的相关方法

```java
    // queue 队列名
    // exchange 交换机名
    // routingKey 用来绑定队列和交换机的路由键
    Queue.BindOk queueBind(String queue, String exchange, String routingKey) throws IOException;

    Queue.BindOk queueBind(String queue, String exchange, String routingKey, Map<String, Object> arguments) throws IOException;

    void queueBindNoWait(String queue, String exchange, String routingKey, Map<String, Object> arguments) throws IOException;
```

## 3.7 队列接触绑定

```java
    Queue.UnbindOk queueUnbind(String queue, String exchange, String routingKey) throws IOException;

    Queue.UnbindOk queueUnbind(String queue, String exchange, String routingKey, Map<String, Object> arguments) throws IOException;
```

## 3.8 交换机绑定的相关方法

```java
    // 该方法将交换机与交换机进行绑定
    // 将 destination 交换机与 source 交换机进行绑定
    // 消息到达 destination 后，destination 会根据 routingKey
    // 将消息发到 source
    Exchange.BindOk exchangeBind(String destination, String source, String routingKey) throws IOException;

    Exchange.BindOk exchangeBind(String destination, String source, String routingKey, Map<String, Object> arguments) throws IOException;

    void exchangeBindNoWait(String destination, String source, String routingKey, Map<String, Object> arguments) throws IOException;
```

# 4 发送消息

```java
    void basicPublish(
            // 交换机名
            String exchange, 
            // 路由键
            String routingKey,
            // mandatory 参数在 basicPublish 的时候设置
            // 1. mandatory 设置为 false，则如果发送到 exchange 的消息没有被路由则丢掉
            // 2. mandatory 设置为 true，则如果发送到 exchange 的消息没有被路由则返回给客户端
            boolean mandatory, 
            // immediate 参数在 rabbitmq 3.x 版本已经废弃了
            // 1. immediate 参数设置为 true，如果交换机在将
            // 消息路由到队列时发现队列上并不存在任何消费者
            // 那么这条消息将会通过 Basic.Return 返回
            // 2. immediate 参数设置为 false，交换机会将消息
            // 路由到队列，不论是否存在消费者
            boolean immediate, 
            // 消息的基本属性集
            AMQP.BasicProperties props, 
            // 真正需要被发送的消息
            byte[] body
    ) throws IOException;
```

以下是一个完整的发送消息的 demo

```java
    private static final ConnectionFactory connectionFactory = new ConnectionFactory() {{
        setHost(IP_ADDRESS);
        setPort(PORT);
        setCredentialsProvider(new DefaultCredentialsProvider(USERNAME, PASSWORD));
    }};
    public static void publishMessage2() {
        try (
                Connection connection = connectionFactory.newConnection();
                Channel channel = connection.createChannel();
        ) {
            // 创建一个 type = "topic"、持久化的、非自动删除的交换器
            channel.exchangeDeclare(
                EXCHANGE_NAME, BuiltinExchangeType.TOPIC.getType(), true, false, null);
            // 创建一个持久化的、非排他的、非自动删除的队列
            channel.queueDeclare(QUEUE_NAME, true, false, false, null);
            // 将交换器与队列通过路由键绑定
            channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, ROUTING_KEY);
            // 发送一条持久化的消息：hello world!
            String message = "hello world";
            int num = 1;
            channel.addReturnListener((replyCode, replyText, exchange, routingKey, properties, body) -> {
                System.out.println("Basic.Return: " + new String(body));
            });
            while (true) {
                channel.basicPublish(
                        EXCHANGE_NAME,
                        ROUTING_KEY,
                        true,
                        true,
                        MessageProperties.PERSISTENT_TEXT_PLAIN,
                        ((num++) + " : " + message).getBytes()
                );
                TimeUnit.SECONDS.sleep(1);
            }
        } catch (TimeoutException | IOException | ShutdownSignalException | InterruptedException e) {
            e.printStackTrace();
        }
    }
```

# 5 消费消息

消费消息有两种模式，一种是推模式，另一种是拉模式

## 5.1 推模式

```java
    String basicConsume(
            // 队列名
            String queue, 
            // 自动 ack，设置为 false 则不自动 ack
            // 这样的好处可以防止客户端接收到消息后
            // 因为异常导致的消息丢失，在客户端处理
            // 完消息之后再 ack
            boolean autoAck, 
            // 消费者标签，用来区分多个消费者
            String consumerTag,
            // 设置为 true 则表示不能将同一个 Connection 中
            // 生产者发送的消息传送给这个 Connection 中的消
            // 费者
            boolean noLocal, 
            // 是否排他性
            boolean exclusive,
            // 设置消费者的其他参数
            Map<String, Object> arguments, 
            // 设置消费者的回掉函数，用来处理 RabbitMQ 推送
            // 归来的消息，比如 DefaultConsumer，使用时需要
            // 覆写其中的方法
            Consumer callback
    ) throws IOException;

    // 如下是回掉方法的一个例子
    DefaultConsumer consumer = new DefaultConsumer(channel) {
        @Override
        // 处理 RabbitMQ 推送过来的消息
        public void handleDelivery(
            String consumerTag, 
            Envelope envelope,
            AMQP.BasicProperties properties, 
            byte[] body
        ) throws IOException {
            System.out.println("recv message: " + new String(body));
            channel.basicAck(envelope.getDeliveryTag(), false);
        }
    };
```

以下是一个完整的推模式的 Demo

```java
    public static void pushMessage() {
        Address[] addresses = {new Address(IP_ADDRESS, PORT)};
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setUsername(USERNAME);
        connectionFactory.setPassword(PASSWORD);
        // 这里的连接方式与生产者的 demo 略有不同，注意辨别区别
        try (
                // 创建连接
                Connection connection = connectionFactory.newConnection(addresses);
                // 创建信道
                final Channel channel = connection.createChannel();
        ) {
            // 设置客户端最多接收未被 ack 的消息的个数
            channel.basicQos(64);
            DefaultConsumer consumer = new DefaultConsumer(channel) {
                @Override
                public void handleDelivery(String consumerTag, Envelope envelope,
                                           AMQP.BasicProperties properties, byte[] body) throws IOException {
                    System.out.println("recv message: " + new String(body));
                    channel.basicAck(envelope.getDeliveryTag(), false);
                }
            };
            channel.basicConsume(QUEUE_NAME, consumer);
            TimeUnit.DAYS.sleep(1);
        } catch (TimeoutException | IOException | InterruptedException e) {
            e.printStackTrace();
        }
    }
```

## 5.2 拉模式

```java
    // queue 队列名
    // autoAck 是否自动回复
    GetResponse basicGet(String queue, boolean autoAck) throws IOException;
```

以下是一个完整的拉模式的 Demo，注意，拉模式不能在循环里使用，否则严重影响性能，如果要在循环里使用，请使用推模式。

```java
    public static void getMessage() {
        Address[] addresses = {new Address(IP_ADDRESS, PORT)};
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setUsername(RabbitProducer.USERNAME);
        connectionFactory.setPassword(RabbitProducer.PASSWORD);
        // 这里的连接方式与生产者的 demo 略有不同，注意辨别区别
        try (
                // 创建连接
                Connection connection = connectionFactory.newConnection(addresses);
                // 创建信道
                final Channel channel = connection.createChannel();
        ) {
            GetResponse response = channel.basicGet(QUEUE_NAME, false);
            System.out.println(new String(response.getBody()));
            channel.basicAck(response.getEnvelope().getDeliveryTag(), false);
        } catch (TimeoutException | IOException e) {
            e.printStackTrace();
        }
    }
```

# 6 消费端的确认与拒绝

## 6.1 消息的确认

RabbitMQ 提供了消息确认机制，当 autoAck 为 false 时，在消息发给消费者之后，RabbitMQ 会为消息打上删除标记，当收到 ack 之后才删除，如果未收到 ack 则等待，等待时常没有限制，但如果消费者与 RabbitMQ 断开了连接，则该消息会返回队列被重新发送。

在 Web 页面上看到的 Ready 和 Unacknowledged 分别对应等待投递给消费者的消息数和已经投递给消费者但是未收到确认信号的消息数。

![web 页面看到的 Ready 和 Unacknowledged](/media/hovel/54.png)

```java
    // 消息确认可以调用 channel 中的这个方法
    // deliveryTag 可以看作消息的编号，它是一个 64 位的长整型值
    // multiple 批量确认，如果为 true ，则执行批量确认，此 deliveryTag 
    // 与此前收到的消息全部确认；如果为 false，则只对当前收到的消息进行
    // 确认
    void basicAck(long deliveryTag, boolean multiple) throws IOException;
```

## 6.2 消息的拒绝

```java
    // 只能拒绝一条
    // requeue 拒绝后是否重新放回队列
    void basicReject(long deliveryTag, boolean requeue) throws IOException;

    // 下面命令可以实现批量拒绝
    // multiple 批量拒绝，如果为 true，则执行批量拒绝，此 deliveryTag 与
    // 此前收到的消息全部拒绝；如果为 false 则只对当前收到的消息进行拒绝
    // requeue 拒绝后是否重新放回队列
    void basicNack(long deliveryTag, boolean multiple, boolean requeue) throws IOException; 
```

# 7 关闭连接

```java
    channel.close();
    // connection 关闭的时候，注册在其上的 channel 也会关闭
    connection.close();

    // 在 Connection 和 Channel 中有个很有用的方法
    // 这个方法可以让我们直到关闭的原因
    void addShutdownListener(ShutdownListener listener);
```

# 参考资料

1. [RabbitMQ 实战指南](https://gitee.com/zhixiangyuan/bookStorage/blob/master/%E7%BC%96%E7%A8%8B/RabbitMQ%20%E5%AE%9E%E6%88%98%E6%8C%87%E5%8D%97.pdf)