---
title: "使用命令行脚本操作 kafka 小记"
date: 2020-02-16T22:25:01+08:00
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

```shell
# 启动命令
$> bin/kafka-server-start.sh <$config_file_path>
## demo: 启动命令
$> bin/kafka-server-start.sh config/server.properties

# 终止命令
$> bin/kafka-server-stop.sh <$config_file_path>
## demo: 终止命令
$> bin/kafka-server-stop.sh config/server.properties

# 创建主题
$> bin/kafka-topics.sh --create --zookeeper <$ip>:<$port> --replication-factor <$num> --partitions <$num> --topic <$topicNname>
## demo: 创建主题
$> bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic Hello-Kafka

# 修改主题分区数
$> bin/kafka-topics.sh --zookeeper <$ip>:<$port> --alter --topic <$topicName> --partitions <$num>
## demo: 修改主题分区数
$> bin/kafka-topics.sh --zookeeper localhost:2181 --alter --topic Hello-Kafka --partitions 8

# 删除主题
# 注意：如果 delete.topic.enable 未设置为 true，则此操作不会产生任何影响
$> bin/kafka-topics.sh --zookeeper <$ip>:<$port> --delete --topic <$topicName> 
## demo: 删除主题
$> bin/kafka-topics.sh --zookeeper localhost:2181 --delete --topic Hello-Kafka

# 查看主题列表
$> bin/kafka-topics.sh --list --zookeeper <$ip>:<$port>
## demo: 查看主题列表
$> bin/kafka-topics.sh --list --zookeeper localhost:2181

# 启动命令行生产者，需要发送的消息在命令行输入
$> bin/kafka-console-producer.sh --broker-list <$ip>:<$port> --topic <$topicName>
## demo: 生产消息
$> bin/kafka-console-producer.sh --broker-list localhost:9092 --topic Hello-Kafka

# 启动命令行消费者
$> $> bin/kafka-console-consumer.sh --zookeeper <$ip>:<$port> --bootstrap-server <$ip>:<$port> --topic <$topicName> --from-beginning
## demo: 消费消息
$> bin/kafka-console-consumer.sh --zookeeper localhost:2181 --bootstrap-server localhost:9092 --topic Hello-Kafka --from-beginning
```