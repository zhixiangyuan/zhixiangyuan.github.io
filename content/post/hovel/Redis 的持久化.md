---
title: "Redis 的持久化"
date: 2019-10-12T16:18:42+08:00
keywords: []
description: ""
tags: [
    "redis"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

Redis 的持久化机制分为两种，RDB 和 AOF

# 1 RDB

RDB 是将当前进程数据生成快照保存到硬盘的过程，触发 RDB 持久化过程分为手动触发和自动触发。

## 1.1 触发机制

### 1.1.1 手动触发

手动触发的命令有两个 save 和 bgsave 命令

save 命令会阻塞 redis 服务器，在生产上禁用

bgsave 是对 save 的优化，通过 fork 操作创建子进程，然后由子进程完成持久化

### 1.1.2 自动触发

1. 使用 save 相关配置，`save <m> <n>` 表示在 m 秒内数据集存在 n 次修改时，自动触发 bgsave
2. 如果从节点执行全量复制操作，主节点自动执行 bgsave 生成 RDB 文件并发送给从节点
3. 执行 debug reload 命令重新加载 Redis 时，也会自动触发 save 操作
4. 默认情况下执行 shutdown 命令是，如果没有开启 AOF 持久化功能则自动执行 bgsave

```shell
$redis-cli> config set save "100 60"
```

## 1.2 RDB 的优缺点

### 1.2.1 优点

1. RDB 是一个紧凑压缩的二进制文件，代表 Redis 在某个时间点上的数据快照。非常适用于备份，全量复制等场景。比如每 6 小时执行 bgsave 备份， 并把RDB文件拷贝到远程机器或者文件系统中（如hdfs），用于灾难恢复。
2. Redis 加载 RDB 恢复数据远远快于 AOF 的方式。

### 1.2.2 缺点

1. RDB 方式数据没办法做到实时持久化/秒级持久化。因为 bgsave 每次运行都要执行 fork 操作创建子进程，属于重量级操作，频繁执行成本过高。
2. RDB 文件使用特定二进制格式保存，Redis 版本演进过程中有多个格式的 RDB 版本，存在老版本Redis 服务无法兼容新版 RDB 格式的问题。
3. 针对 RDB 不适合实时持久化的问题，Redis 提供了 AOF 持久化方式来解决。

# 2 AOF

AOF 默认不开启，开启该功能需要配置 `appendonly yes`，AOF 文件名通过 appendfilename 配置设置，默认文件名是 appendonly.aof。

```shell
$redis-cli> config set appendonly yes
```

![AOF 工作流程](/media/hovel/41.png)

1. 所有的写入命令会追加到 aof_buf 缓冲区中
2. AOF 缓冲区根据对应的策略向硬盘做同步操作
3. 随着 AOF 文件越来越大，需要定期对 AOF 文件进行重写，达到压缩文件的目的
4. 当 Redis 服务器重启时，可以加载 AOF 进行数据恢复

下面单独介绍每一步

## 2.1 命令写入

写入到 aof_buf 中的就是 redis 的定制协议格式

比如说 `set hello world` 写入到硬盘就是 `*3\r\n$3\r\nset\r\n$5\r\nhello\r\n$5\r\nworld\r\n`

### 2.1.1 为什么要写入到缓冲区而不是硬盘

1. 写入到硬盘太慢了，可能会造成阻塞
2. 可以使用不同的策略做缓冲区同步

## 2.2 文件同步

文件同步缓冲区中的策略

![缓冲区的同步策略](/media/hovel/42.png)

```shell
# 写入缓冲区后立即写入硬盘
$redis-cli> config set appendfsync always
# 每过一秒写入一次
$redis-cli> config set appendfsync everysec
# 由操作系统调度，通常 30s 一次
$redis-cli> config set appendfsync no
```

## 2.3 重写机制

随着命令不断写入 AOF，文件会越来越大，为了解决这个问题，Redis 引入 AOF 重写机制压缩文件体积。AOF 文件重写是把当前 Redis 进程内的数据转化为写命令同步到新 AOF 文件的过程。

重写过后文件变小是由于以下原因

1. set 和 del 命令抵消了
2. 多条 push 命令可以合并

重写的触发也有两种方式，一种是手动触发，另一种是自动触发

1. 手动触发：直接调用 bgrewriteaof 命令
2. 自动触发：根据 auto-aof-rewrite-min-size 和 auto-aof-rewrite-percentage 参数确定自动触发时机。
    - auto-aof-rewrite-min-size：表示运行 AOF 重写时文件最小体积，默认为 64MB。
    - auto-aof-rewrite-percentage：代表当前AOF文件空间 （aof_current_size）和上一次重写后AOF文件空间（aof_base_size）的比值。

```shell
# 触发 aof 持久化
$redis-cli> bgrewriteaof
# 设置 aof 自动持久化的配置
$redis-cli> config set auto-aof-rewrite-min-size 100MB
$redis-cli> config set auto-aof-rewrite-percentage 10
```

# 参考资料

1.[Redis 开发与运维](https://gitee.com/zhixiangyuan/bookStorage/raw/master/%E7%BC%96%E7%A8%8B/Redis%20%E5%BC%80%E5%8F%91%E4%B8%8E%E8%BF%90%E7%BB%B4.pdf)