---
title: "Redis 慢查询相关配置与命令"
date: 2019-10-09T17:54:45+08:00
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

# 1 相关配置

配置有两个，分别是 `slowlog-log-slower-than` 和 `slowlog-max-len`，两个配置的含义如下

- `slowlog-log-slower-than`: 超过这个时间的查询为慢查询，时间单位为微秒
- `slowlog-max-len`: 保存的慢查询的条数

设置该配置的命令
```shell
# 设置慢查询的时间为大于 20000 微秒，也就是 20 毫秒
$redis-cli> config set slowlog-log-slower-than 20000
# 设置保存慢查询的条数为 1024 条
$redis-cli> config set slowlog-max-len 1024
# 将配置写入配置文件，否则下次重启配置就不生效了
$redis-cli> config rewrite
```

# 2 相关命令

## 2.1 获取慢查询日志

```shell
# n 为获取的慢查询的条数
$redis-cli> slowlog get <n>

# 下面是获取到的部分数据

...
# 35 是第三十五条数据
 35)    
  # 1) 2) 3) 4) 表示四个属性
  # 标识 id
  1)   "105305" 
  # 发生时的时间戳 
  2)   "1568181878"
  # 命令耗时
  3)   "17056"
  # 执行命令和参数
  4)      
        # 执行的命令
        1)    "DEL"
        # 参数
        2)    "monitor:mindray_n_series:miss_ots_data:"
...
```

## 2.2 获取慢查询日志列表当前长度

```shell
$redis-cli> slowlog len
```

## 2.3 清除所有慢查询日志

```shell
$redis-cli> slowlog reset
```

# 3 最佳实践

## 3.1 slowlog-max-len 配置建议

线上建议调大慢查询列表记录慢查询时 redis 会对长命令做截断操作，并不会占用大量内存。增大慢查询列表可以减缓慢查询被剔除的可能，线上建议设置 1000 以上。

## 3.2 slowlog-log-slower-than 配置建议

默认值超过 10 毫秒判定为慢查询，需要根据 redis 并发亮调整该值，由于 redis 采用单线程响应命令，对于高流量的场景，如果命令执行时间在 1 毫秒以上，那么 redis 最多可支撑 OPS 不到 1000.因此对于高 OPS 场景的 redis 建议设置为 1 毫秒。

注：OPS 即 operation per second 每秒操作次数

## 3.3 其他注意事项

慢查询只记录命令执行时间，并不包括命令排队和网络传输的时间。因此客户端执行命令的时间会大于命令实际执行时间。因为命令执行排队机制，慢查询会导致其他命令级联阻塞，因此当客户端出现请求超时，需要检查该时间点是否有对应的慢查询，从而分析出是否为慢查询导致的命令级联阻塞。

由于慢查询日志是一个先进先出的队列，也就是说如果慢查询比较多的情况下，可能会丢失部分慢查询命令，为了防止这种情况发生，可以定期执行 slow get 命令将慢查询日志持久化到其他存储中（例如 Mysql），然后可以制作可视化界面进行查询。像 Redis 私有云 CacheCloud 已经提供了这样的功能。

# 参考资料

1.[Redis 开发与运维](https://gitee.com/zhixiangyuan/bookStorage/raw/master/%E7%BC%96%E7%A8%8B/Redis%20%E5%BC%80%E5%8F%91%E4%B8%8E%E8%BF%90%E7%BB%B4.pdf)