---
title: "Redis 提供的命令行工具"
date: 2019-10-09T18:33:21+08:00
keywords: []
description: ""
tags: [

]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

# 1 redis-server

```shell
# 直接运行
$> redis-server

# 可以在后面指定配置然后运行
$> redis-server --configKey1 configValue1 --configKey2 configValue2

# 比如说修改端口号
$> redis-server --port 4500

# 指定配置文件启动
$> redis-server /etc/redis.conf

# 以守护进程的方式启动 redis
$> redis-server --daemonize [yea|no]

# 测试是否操作系统是否能稳定的分配指定的内存给 redis
$> redis-server --test-memory 1024
```

| 配置名    | 配置说明                                   |
| --------- | ------------------------------------------ |
| port      | 端口                                       |
| logfile   | 日志文件                                   |
| dir       | Redis 工作目录（存放持久化文件和日志文件） |
| daemonize | 是否以守护进程的方式启动 redis             |

# 2 redis-cli

注：cli 即 Command Line Interface

redis-cli 用于操作 redis 服务，操作 redis 的方式有两种，一种是交互式方式，另一种是命令方式

```shell
# 交互式方式
## 执行以下命令则进入交互式命令行，后面的输入的命令能直接对 redis 产生效果
$> redis-cli -h 127.0.0.1 -p 6379
$>127.0.0.1:6378> set hello world
OK
$>127.0.0.1:6379> get hello
"world"

# 命令方式
$> redis-cli  -h 127.0.0.1 -p 6379 get hello
"world"

# 注：如果参数不带 -h 127.0.0.1 则地址默认是 127.0.0.1
#     如果参数不带 -p 6379 则端口默认 6379

# 停止 redis 服务
$> redis-cli shutdown
# shutdown 命令有一个参数，是否在关闭前持久化
$> redis-cli shutdown [nosave|save]
```

## 2.1 recis-cli 命令的参数

```shell
# 1. 
# -r (repeat) 代表将命令执行几次 -r 3 表示重复三次
$> redis-cli -r 3 ping
PONG
PONG
PONG

# 2. 
# -i (interval) 代表每隔几秒执行一次命令
# -i 参数必须和 -r 参数一起使用
# 如下命令表示 recis-cli ping 每隔一秒执行一次，总共执行三次
$> redis-cli -r 3 -i 1 ping
# -i 0.01 表示以 10 毫秒为间隔执行
$> redis-cli -r 3 -i 0.01 ping

# 3. 
# -x 参数会从标准输入（stdin）读取数据作为 redis-cli 的最后一个参数
# 下面这个例子等同于 $> redis-cli set hello world
$> echo "world" | redis-cli -x set hello

# 4. 
# -c（cluster）选项是连接 redis cluster 节点时使用的
# -c 选项可以防止 moved 和 ask 异常

# 5. 
# -a 如果 redis 配置了密码，可以用 -a（auth）选项，这样就不需要手动输入 auth 命令了

# 6. 
# --scan 和 --pattern 用于扫描指定模式的键，相当于使用 scan 命令

# 7. 
# --rdb 这个选项会请求 redis 实例生成并发送 RDB 持久文件保存在本地。
# 可使用这个选项做持久化文件的定期备份

# 8.
# --pipe 选项用于将命令封装成 redis 通信协议定义的数据格式，批量发送给 redis 执行

# 9.
# --bigkeys 选项使用 scan 命令对 redis 的键进行采样，从中找到内存占用比较大的键值
# 这些键可能是系统的平静瓶颈

# 10.
# --eval 选项用于执行指定 Lua 脚本
```

```shell
# 11. latency 有三个选项分别是 --latency、--latency-history、--latency-dist
#  * --latency 该选项可以用于测试客户端到目标 redis 的网络延迟
# 假设网络拓扑图如下所示，分别在客户端 A 和 B 测试该命令的效果如下
# 客户端 B
$> redis-cli -h (machineB) --latency
min: 0, max: 1, avg: 0.07 (4211 samples)
# 客户端 A
$> redis-cli -h (machineA) --latency
min: 0, max: 2, avg: 1.04 (2096 samples)
# 可以看到由于客户端 A 离 B 远，所以延迟较高
```

![客户端 A 与 客户端 B](/media/hovel/37.png)

```shell
# 12. 
# --stat 选项可以实时获取 redis 的重要统计信息，虽然 info 命令中的统计
# 信息更全，但是 --stat 可以看到一些增量数据（例如 requests）对于 redis 
# 的运维有一定的帮助

# 13.
# --raw 和 --no-raw 分别是返回格式化之后的内容和返回原始内容
$>redis-cli set hello "你好"
$>redis-cli --raw get hello
你好
$>redis-cli --no-raw get hello
"\xe4\xbd\xa0\xe5\xa5\xbd"
```

# 3 redis-benchmark

redis-benchmark 可以为 redis 做基准性能测试，它提供了很多选项帮助开发和运维人员测试 redis 的相关性能

```shell
# 1. -c 
# -c（client）选项代表客户端的并发数量（默认是 50）

# 2. -n<requests>
# -n（num）选项代表客户端请求总量（默认是 100000）
# 如下命令代表 100 个客户端同时请求 redis 一共执行 20000 次
# redis-benchmark 会对各类数据结构的命令进行测试，并给出性能指标
$> redis-benchmark -c 100 -n 20000

# 3. -q
# -q 选项仅仅显示 redis-benchmark 的 request per second 的信息

# 4. -r
# 向 redis 中随机插入一些键

# 5. -P
# -P 代表每个请求 pipeline 的数据量（默认为 1）

# 6. -k
# -k 选项代表客户端是否使用 keepalive，1 为使用，0 为不使用，默认值为 1

# 7. -t
# -t 选项可以对指定命令进行基准测试
$> redis-benchmark -t get,set -q

# 8. --csv
# --csv 选项会将结果按照 csv 格式输出，便于后续处理，如到处到 Excel
```

# 参考资料

1. [《Redis 开发与运维》](#)