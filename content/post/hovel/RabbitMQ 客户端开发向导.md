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

# 3 使用交换器和队列

使用交换器与队列之前需要先声明，如果声明的一个已存在的交换器，如果交换器参数完全匹配则什么都不做，如果参数不匹配则会抛出异常。

## 3.1 exchangeDeclare 方法详解

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
    // 这个方法会判断交换机是否存在，如果存在则正常返回，如果不存在则抛出异常同时 Channel 也会被关闭。
    Exchange.DeclareOk exchangeDeclarePassive(String name) throws IOException;
```

