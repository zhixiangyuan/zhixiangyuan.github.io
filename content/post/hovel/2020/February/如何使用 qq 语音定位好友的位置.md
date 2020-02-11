---
title: "如何使用 qq 语音定位好友的位置"
date: 2020-02-11T22:50:10+08:00
keywords: []
description: ""
tags: [
    "M78 星云"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
markup: mmark
mathjax: false
draft: true
---

首先，这里的原理是和好友语音，那么会首先从腾讯服务器获取到对方的 IP 地址，然后再本地连线对方，那么我们在这里进行抓包便可以知道对方的 IP 地址。抓包时使用 wireshark 进行抓包，抓包条件为 `udp.length==80` 过滤出所有的包，之后会有大量的 udp 包，当出现有来有回的 udp 包，记下该地址，该地址就是对方的 IP 地址。

在查找对方位置的时候使用接口根据 IP 来查找对方的经纬度，然后在 google 地图上输入经纬度，确定对方的具体位置。

```
下面是一些可以用 IP 查找经纬度的网站

1. https://ip-api.com/ 
    - lat 是纬度
    - lon 是经度
    - 备注：用我自己的 IP 测过，到市是准的，但是县就歪了
2. http://www.chacuo.net/
    - 直接可以查出来位置，比上面准，感觉误差在 1000 米
3. https://lbs.amap.com/api/webservice/guide/api/ipconfig/
    - 高德的 api，有免费的 demo 可以用，不过不提供到人的精确经纬度
4. https://www.amap.com/service/locator?rand=0.4097587599713388
    - 高德地图的官网查询接口，会返回经纬度
    - 请求的时候加上 cookie 就行，至于 cookie 是怎么放进去的等等问题还需要研究
    - 官网地址：https://www.amap.com/
```