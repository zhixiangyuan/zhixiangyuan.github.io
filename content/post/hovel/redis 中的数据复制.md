---
title: "Redis 中的数据复制"
date: 2019-10-14T11:19:41+08:00
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

# 1 数据复制

## 1.1 建立复制

在从节点的配置文件中加入以下配置，然后启动即可

`slaveof <masterip> <masterport>`

启动过后通过 `info replication` 查看节点信息

```shell
$redis_master_node> info replication
# Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=6380,state=online,offset=29,lag=1
master_repl_offset:29
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:28
$redis_slave_node> info replication
# Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=6380,state=online,offset=29,lag=1
master_repl_offset:29
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:28
```

除了配置文件外也可以通过以下两个方法进行配置

```shell
# 在启动服务时指定
$>redis-server --slaveof <masterip> <masterport>

# 在启动之后指定
# 如果使用此命令从一个主节点切换到另外一个主节点
# 那么当前节点会删除所有数据，然后从新的主节点上同步数据
$redis_cli> slaveof <masterip> <masterport>
```

### 1.1.1 测试案例

```shell
# 以下是建立复制之后的效果
$redis_master_node> get hello
(nil)
$redis_slave_node> get hello
(nil)
# master 节点设置值
$redis_master_node> set hello world
OK
$redis_master_node> get hello
"world"
# 主从同步成功
$redis_slave_node> get hello
"world"
```

## 1.2 断开复制

使用 slaveof 命令来断开与主节点的复制关系，断开之后，从节点晋升为主节点，从节点断开复制之后不会抛弃原有数据，仅是无法再获取主节点上的数据变化

```shell
$redis_slave_node> info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:up
master_last_io_seconds_ago:5
master_sync_in_progress:0
slave_repl_offset:14618
slave_priority:100
slave_read_only:1
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
$redis_slave_node> slaveof no one
OK
$redis_slave_node> info replication
# Replication
role:master
connected_slaves:0
master_repl_offset:14702
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```

## 1.3 安全性

对于数据比较重要的节点，主节点会通过设置 requirepass 参数进行密码验证，这时所有的客户端访问必须使用 auth 命令实行校验。从节点与主节点的复制连接是通过一个特殊标识的客户端来完成，因此需要配置从节点的 masterauth 参数与主节点密码保持一致，这样从节点才可以正确地连接到主节点并发起复制流程。

## 1.4 只读

默认情况下，从节点使用 slave-read-only=yes 配置为只读模式。由于复制只能从主节点到从节点，对于从节点的任何修改主节点都无法感知，修改从节点会造成主从数据不一致。因此建议线上不要修改从节点的只读模式。

## 1.5 传输延迟

主从节点一般部署在不同机器上，复制时的网络延迟就成为需要考虑的问题，redis 为我们提供了 repl-disable-tcp-nodelay 参数用于控制是否关闭 TCP_NODELAY，默认关闭，说明如下：

- 当关闭时，主节点产生的命令数据无论大小都会及时地发送给从节点，这样主从之间延迟会变小，但增加了网络带宽的消耗。适用于主从之间的网络环境良好的场景，如同机架或同机房部署。

- 当开启时，主节点会合并较小的TCP数据包从而节省带宽。默认发送时间间隔取决于 Linux的内核，一般默认为 40 毫秒。这种配置节省了带宽但增大主从之间的延迟。适用于主从网络环境复杂或带宽紧张的场景，如跨机房部署。

# 2 拓扑结构

## 2.1 一主一从结构

![一主一从结构](/media/hovel/43.png)

使用此结构可以只在从节点上开启 AOF，这样避免了持久化对主节点性能的影响，不过这样用有一个需要注意的地方，重启主节点之前要先断开从节点，不然主节点一启动什么数据都没有，从节点也复制过去把自己给清空了，从节点就失去了意义。

## 2.2 一主多从结构

![一主多从结构](/media/hovel/44.png)

一主多从结构（又称为星形拓扑结构）使得应用端可以利用多个从节点实现读写分离。对于读占比较大的场景，可以把读命令发送到从节点来分担主节点压力。同时在日常开发中如果需要执行一些比较耗时的读命令，如：keys、sort等，可以在其中一台从节点上执行，防止慢查询对主节点造成阻塞从而影响线上服务的稳定性。对于写并发量较高的场景，多个从节点会导致主节点写命令的多次发送从而过度消耗网络带宽，同时也加重了主节点的负载影响服务稳定性。

## 2.3 树状主从结构

![树状主从结构](/media/hovel/45.png)

树状主从结构（又称为树状拓扑结构）使得从节点不但可以复制主节点数据，同时可以作为其他从节点的主节点继续向下层复制。通过引入复制中间层，可以有效降低主节点负载和需要传送给从节点的数据量。如图所示，数据写入节点 A 后会同步到 B 和 C 节点，B 节点再把数据同步到 D 和 E 节点，数据实现了一层一层的向下复制。当主节点需要挂载多个从节点时为了避免对主节点的性能干扰，可以采用树状主从结构降低主节点压力。
