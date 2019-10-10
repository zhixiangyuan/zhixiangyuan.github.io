---
title: "Redis 中的 GEO 功能"
date: 2019-10-10T15:38:27+08:00
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
draft: true
---

通过 GEO 功能可以实现诸如附近位置、摇一摇这类依赖于地理位置信息的功能，对于需要实现这些功能的开发者来说是一大福音。

# 1 命令

## 1.1 添加地理位置信息

`geoadd <key> <longitude> <member> [key longitude member...]`

longitude、latitude、member 分别是该地理位置的经度、纬度、成员

下表展示了五个城市的经纬度

| 城市   | 经度   | 纬度  | 成员         |
| ------ | ------ | ----- | ------------ |
| 北京   | 116.28 | 39.55 | beijin       |
| 天津   | 117.12 | 39.08 | tianjin      |
| 石家庄 | 114.29 | 38.02 | shijiazhuang |
| 唐山   | 118.01 | 39.38 | tangshan     |
| 保定   | 115.29 | 38.51 | baoding      |

```shell
$redis-cli> geoadd cities:locations 116.28 39.55 beijin
(integer) 1
$redis-cli> geoadd cities:locations 117.12 39.08 tianjin
(integer) 1
$redis-cli> geoadd cities:locations 114.29 38.02 shijiazhuang
(ingeger) 1
$redis-cli> geoadd cities:locations 118.01 39.38 tangshan
(integer) 1
$redis-cli> geoadd cities:locations 115.29 38.51 baoding
(integer) 1

# 也可以用一条命令一次性添加进去
$redis-cli> geoadd cities:locations 116.28 39.55 beijin 117.12 39.08 tianjin 114.29 38.02 shijiazhuang 118.01 39.38 tangshan 115.29 38.51 baoding
```

## 1.2 获取地理位置信息

`geopos <key> <member> [member...]`

```shell
# 获取天津的经纬度
$redis-cli> geopos cities:locations tianjin
1) 1) "117.12000042200088501"
   2) "39.0800000535766543"
```

## 1.3 获取两个地理位置的距离


`geodist <key> <member1> <member2> <unit>`

计算 member1 与 member2 之间的距离，unit 有四种可选：

1. m（meters）米
2. km（kilometers）公里
3. mi（miles）英里
4. ft（feet）尺

```shell
# 计算天津到北京的距离，以公里为单位
$redis-cli> geodist cities:locations tianjin beijing km
"91.3106"
```

## 1.4 获取指定范围内的地理位置信息集合

