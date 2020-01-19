---
title: "如何在 MySQL 中快速生成一张亿级数据量的表"
date: 2020-01-16T09:08:28+08:00
keywords: []
description: ""
tags: [
    "MySQL"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

>本文使用的测试硬件是树莓派4B 4G 版本

在网上找了一下，发现有两种方法，一种是弄一张内存表，用存储过程  向内存表中插入数据，然后再将数据写回数据表，实际测试效果并不理想。另一种比较靠谱，直接查询某张表，然后在 SELECT 中生成数据或者直接查出数据，然后插入到需要数据的表，这种效果比较理想。所以下面演示一下第二种方法。

```mysql
# 表结构
CREATE TABLE `film` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(100) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

# 我这张表目前有 7,271,131 行
INSERT INTO film ( `name` ) SELECT Rand() FROM film;

# 下面是插入耗时，可以看到七百多万的数据 185 秒搞定了，很棒
Affected rows: 7271131, Time: 185.557000s
```

对于生成随机数可以使用 `rand()` 函数，`rand()` 函数会生成一个 [0,1) 范围之间的小数，我们可以对它乘以 10 的倍数然后调用 `round()` 来获取到相应的整数，例如 `round(rand() * 10)`。`round()` 是一个四舍五入的函数。如果我们需要获取字符串可以考虑使用 `MD5(rand())` 来获取一个随机 md5 值。

对于 TIME、DATE、DATETIME、TIMESTAMP、YEAR 这种时间类型，可以通过 now() 生成时间。
