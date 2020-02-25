---
title: "kafka 发送消息分区选择策略"
date: 2020-02-18T13:51:46+08:00
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

# 1 大体逻辑

partition 的分区选择发生在发送数据的生产者上面，在选择分区时分为两种情况

- 消息的 key 为 null

那么则获取该 topic 的一个随机数，然后判断 topic 的可用分区数是否大于 0，如果大于 0 则使用获取的随机数进行取模，取模的值便是分区号。如果 topic 的可用分区数小于等于 0，则用获取的随机数和总分区数进行取模运算（也就是随机选择一个不可用的分区）。

- 消息的 key 不为 null

如果消息的 key 不为 null，那么直接使用 `murmur2` 算法计算出 key 的 hash 值，然后和分区数做取模运算。

# 2 通过源码查看怎么来的

```java
// 从发送消息的 send 方法开始向下找，先进入 doSend 方法，实际的分区代码是下面这句
    int partition = partition(record, serializedKey, serializedValue, cluster);

// 进去之后可以看到下面这段代码
    private int partition(ProducerRecord<K, V> record, byte[] serializedKey, byte[] serializedValue, Cluster cluster) {
        Integer partition = record.partition();
        return partition != null ?
                // 如果该 record 有分区了，那么就直接使用该分区
                partition :
                // 如果没有分区则使用 partitioner 来计算分区，默认使用 DefaultPartitioner.class 类
                partitioner.partition(
                        record.topic(), record.key(), serializedKey, record.value(), serializedValue, cluster);
    }

// 下面接着看 DefaultPartitioner 内部的实现
    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
        List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
        int numPartitions = partitions.size();
        // 判断是否有 key
        if (keyBytes == null) {
            // 获取一个随机数，内部是如果上次有计算过随机数
            // 那么就在上次的随机数上加一
            int nextValue = nextValue(topic);
            List<PartitionInfo> availablePartitions = cluster.availablePartitionsForTopic(topic);
            // 判断当前是否有可用分区
            if (availablePartitions.size() > 0) {
                // 通过 nextValue 这个随机数和可用分区做取模运算
                int part = Utils.toPositive(nextValue) % availablePartitions.size();
                return availablePartitions.get(part).partition();
            } else {
                // 用 nextValue 和总分区数做取模计算处一个不可用的分区
                return Utils.toPositive(nextValue) % numPartitions;
            }
        } else {
            // 使用 murmur2 计算 key 的 hash，然后和总分区数做取模运算
            return Utils.toPositive(Utils.murmur2(keyBytes)) % numPartitions;
        }
    }
```

# 参考文章

1. [kafka 发送消息分区选择策略详解](https://leokongwq.github.io/2017/02/27/mq-kafka-producer-partitioner.html)
   - 讲的很详细