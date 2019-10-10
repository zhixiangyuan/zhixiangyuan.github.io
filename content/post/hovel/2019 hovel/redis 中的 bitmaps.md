---
title: "Redis 中的 Bitmaps"
date: 2019-10-10T12:47:37+08:00
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

# 1 存储 bitmap 的数据结构

其实就是普通的 string，只不过可以通过 bitmap 来操作二进制位为 0 或 1，比如说下面这个例子

```shell
# 先设置一个键值对
$redis-cli>set hello world
# 然后修改 world 的二进制位
# setbit <key name> <offset> <0|1>
$redis-cli>setbit hello 1 0
$redis-cli>get hello
7orld
# 可以看到由于修改了 w 的二进制位，从而让其变成了 7
```

# 2 相关命令

## 2.1 设置值

`$redis-cli> setbit <key> <offset> <value>`

我们将 string 看成一个以 bit 为单位的二进制数组，那么上面的命令就很好理解了，key 就是 string 的 key，offset 是偏移量，比如说 0，那么就表示数组下标为 0 的那个位，value 可能是 0|1，表示该位需要设置的值。

通过这个功能，我们可以实现一个小功能，记录每天用户是否登陆，命令如下

```shell
# 下面这个命令这就表示 user id = 0 的用户今天登陆了应用
$redis-cli> setbit unique:users:2016-04-05 0 1
$redis-cli> setbit unique:users:2016-04-05 10 1
$redis-cli> setbit unique:users:2016-04-05 11 1
```

## 2.2 获取值

`getbit <key> <offset>`

如果查询的 offset 不存在，同样返回 0

## 2.3 获取 bitmaps 指定范围值为 1 的个数

`bitcount <start> <end>`

这里统计时候的区间 [start, end] 它两头都是闭区间，并且这表示的是字节数，比如说 [0, 1] 指的是从第 0 个字节开始到第一个字节，换算成位也就是 [0, 15]。

测试一下效果

```shell
$redis-cli> set unique:users:2016-04-05 7 1
# 内存中位的样子 0b00000001
$redis-cli> set unique:users:2016-04-05 8 1
# 内存中位的样子 0b00000001_10000000
# 使用 bitcount 统计一下看看
$redis-cli> bitcount unique:users:2016-04-05 0 0
1
$redis-cli> bitcount unique:users:2016-04-05 0 1
2
$redis-cli> bitcount unique:users:2016-04-05
2
```

## 2.4 bitmap 间的运算

`bitop <op> <destkey> [key...]`

op 是操作符可选：and（交集）、or（并集）、not（非）、xor（异或）
destkey 是计算完的结果保存在这个 key
key... 是需要进行运算的 key


通过这个计算的功能我们实现一些功能

```shell
# 计算 2016-04-04 和 2016-04-03 两天都访问过网站的用户
$redis-cli> bitop and unique:users:and:2016-04-03_04 unique:users:2016-04-03 unique:users:2016-04-04
# 计算 2016-04-04 和 2016-04-03 任意一天访问过网站的用户
$redis-cli> bitop or unique:users:or:2016-04-03_04 unique:users:2016-04-03 unique:users:2016-04-04
```

## 2.5 计算 bitmaps 中第一个值位 target bit 的偏移量

`bitpos <key> <0|1> <start> <end>`

start 和 end 代表开始字节和结束字节，同样单位是字节

```shell
# 计算 2016-04-03 访问过网站的最小用户 id
$redis-cli> bitpos unique:users:2016-04-03 1
```

# 3 bitmaps 实践

假设网站用户有 1 亿，每天独立访问的用户只有 5 千万，如果每天用集合类型和 bitmaps 分别存储活跃用户可以得到如下表个

| 数据类型 | 每个用户 id 所占空间 | 需要存储的用户量 | 全部内存量                   |
| -------- | -------------------- | ---------------- | ---------------------------- |
| 集合类型 | 64位                 | 50 000 000       | 64 bit * 50 000 000 = 400 MB |
| bitmaps  | 1 位                 | 100 000 000      | 1 bit * 100 000 000 =  12.5 MB |

很显然这种情况使用 bitmap 可以节省很多的内存空间，尤其是随着时间的推移节省的内存是非常可观的。下表是随着时间推移存储空间的对比

|         | 一天    | 一个月 | 一年   |
| ------- | ------- | ------ | ------ |
| set     | 400 MB  | 12 GB  | 144 G  |
| bitmaps | 12.5 MB | 375 MB | 4.5 GB |

但是 bitmaps 并不是银弹，加入该网站每天的独立访问用户很少，比如说只有 10 万，那么两者的对比如下表所示，这个时候使用 bitmaps 就不是很合适，因为大部分的位都是 0

| 数据类型 | 每个用户 id 所占空间 | 需要存储的用户量 | 全部内存量                   |
| -------- | -------------------- | ---------------- | ---------------------------- |
| 集合类型 | 64位                 | 100 000          | 64 bit * 100 000 = 800 KB    |
| bitmaps  | 1 位                 | 100 000 000      | 1 bit * 100 000 000 =  12.5 MB |

# 参考资料

1.[Redis 开发与运维](https://gitee.com/zhixiangyuan/bookStorage/raw/master/%E7%BC%96%E7%A8%8B/Redis%20%E5%BC%80%E5%8F%91%E4%B8%8E%E8%BF%90%E7%BB%B4.pdf)