---
title: "kafka-client 使用示例"
date: 2020-02-17T09:38:22+08:00
keywords: []
description: ""
tags: [
    "kafka"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

# 1 生产者示例

```java
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.Producer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.Future;
import java.util.concurrent.TimeUnit;

public class ProducerMain {

    private static Producer<String, String> createProducer() {
        // 设置 Producer 的属性
        Properties properties = new Properties();
        // 设置 Broker 的地址
        properties.put("bootstrap.servers", "127.0.0.1:9093");
        // 0-不应答。1-leader 应答。all-所有 leader 和 follower 应答。
        properties.put("acks", "1");
        // 发送失败时，重试发送的次数
        properties.put("retries", 3);
//        properties.put("batch.size", 16384);
//        properties.put("linger.ms", 1);
//        properties.put("client.id", "DemoProducer");
//        properties.put("buffer.memory", 33554432);
        // 消息的 key 的序列化方式
        properties.put("key.serializer", StringSerializer.class.getName());
        // 消息的 value 的序列化方式
        properties.put("value.serializer", StringSerializer.class.getName());

        // 创建 KafkaProducer 对象
        // 因为我们消息的 key 和 value 都使用 String 类型，所以创建的 Producer 是 <String, String> 的泛型。
        return new KafkaProducer<>(properties);
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        // 创建 KafkaProducer 对象
        Producer<String, String> producer = createProducer();

        // 同步发送消息
        for (int i = 0; ; i++) {
            // 创建消息。传入的三个参数，分别是 Topic ，消息的 key ，消息的 message
            ProducerRecord<String, String> message = new ProducerRecord<>("TestTopic", "key", "test message : " + i);
            Future<RecordMetadata> sendResultFuture = producer.send(message);
            RecordMetadata result = sendResultFuture.get();
            System.out.println(String.format("message sent to [%s], partition [%s], offset [%s].",
                    result.topic(), result.partition(), result.offset()));
            TimeUnit.MILLISECONDS.sleep(500);
        }
    }
}
```

# 2 消费者示例

```java
import org.apache.kafka.clients.consumer.Consumer;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.serialization.StringDeserializer;

import java.time.Duration;
import java.util.Collections;
import java.util.Properties;

public class ConsumerMain {

    private static Consumer<String, String> createConsumer() {
        // 设置 Producer 的属性
        Properties properties = new Properties();
        // 设置 Broker 的地址
        properties.put("bootstrap.servers", "127.0.0.1:9093");
        // 消费者分组
        properties.put("group.id", "demo-consumer-group");
        // 设置消费者分组最初的消费进度为 earliest 。可参考博客 https://blog.csdn.net/lishuangzhe7047/article/details/74530417 理解
        properties.put("auto.offset.reset", "earliest");
        // 是否自动提交消费进度
        properties.put("enable.auto.commit", true);
        // 自动提交消费进度频率
        properties.put("auto.commit.interval.ms", "1000");
        // 消息的 key 的反序列化方式
        properties.put("key.deserializer", StringDeserializer.class.getName());
        // 消息的 value 的反序列化方式
        properties.put("value.deserializer", StringDeserializer.class.getName());

        // 创建 KafkaProducer 对象
        // 因为我们消息的 key 和 value 都使用 String 类型，所以创建的 Producer 是 <String, String> 的泛型。
        return new KafkaConsumer<>(properties);
    }

    public static void main(String[] args) {
        // 创建 KafkaConsumer 对象
        Consumer<String, String> consumer = createConsumer();

        // 订阅消息
        consumer.subscribe(Collections.singleton("TestTopic"));

        // 拉取消息
        while (true) {
            // 拉取消息。如果拉取不到消息，阻塞等待最多 10 秒，或者等待拉取到消息。
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(10));
            // 遍历处理消息
            records.forEach(record -> System.out.println(String.format("key: [%s], value: [%s]", record.key(), record.value())));
        }
    }
}
```

# 参考文章

1. [芋道 Kafka 安装部署](http://www.iocoder.cn/Kafka/install/?self)