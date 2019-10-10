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
   1) "39.0800000535766543"
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

`georadius <key> <longitude> <latitude> <radiusm|km|ft|mi> [withcoord] [withdist] [withhash] [COUNT count] [asc|desc] [store key] [storedist key]`

`georadiusbymember <key> <member> <radiusm|km|ft|mi> [withcoord] [withdist] [withhash] [COUNT count] [asc|desc] [store key] [storedist key]`

georadius 和 georadiusbymember 两个命令的作用是一样的，都是以一个地理位置为中心算出指定半径内的其他地理位置信息，不同的是 georadius 命令的中心位置给出了具体的经纬度，georadiusbymember 只给出成员即可。

可选参数：

- withcoord： 返回结果中包含经纬度
- withdist：返回结果中包含离中心节点位置的距离
- withhash：返回结果中包含 geohash，有关 geohash 后面介绍
- COUNT count：指定返回结果的数量
- asc|desc：返回结果按照离中心节点的距离做升许或者降序
- store key： 将返回结果的地理位置信息保存到指定键
- storedist key：将返回结果离中心节点的距离保存到指定键 

## 1.5 geohash

`geohash <key> <member> [member...]`

老实说，不明白生成的这个字符串有什么用

```shell
$redis-cli> geohash cities:locations tianjin
"wwgq34k1tb0"
```

## 1.6 删除地理位置信息

`zrem <key> <member>`

GEO 没有提供删除成员的命令，但是因为 GEO 的底层实现是 zset，所以可以借用 zrem 命令实现对地理位置信息的删除。

# 参考资料

1.[Redis 开发与运维](https://gitee.com/zhixiangyuan/bookStorage/raw/master/%E7%BC%96%E7%A8%8B/Redis%20%E5%BC%80%E5%8F%91%E4%B8%8E%E8%BF%90%E7%BB%B4.pdf)