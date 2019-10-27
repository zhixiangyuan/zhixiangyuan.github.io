---
title: "MySQL 三大引擎"
date: 2019-10-27T11:16:36+08:00
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

MySQL 常用的引擎有 InnoDB、MyISAM、Memory，默认是 InnoDB

# 1 InnoDB

这种引擎序列化在磁盘中，支持事务，支持行级锁，B+Tree 索引。拥有良好的 ACID 特性，适用于高并发、更新操作比较频繁、需要事务支持并且对自动灾备有要求的表。

缺点在于读写性能比 MyISAM 差，占用的空间也比较大

# 2 MyISAM 

这种引擎序列化在磁盘中，但是不支持事务，仅支持表级锁，B+Tree 索引

优点是占用空间小，处理速度比 InnoDB 快

缺点是不支持事务的完整性和并发性

# 3 Memory

这种引擎将数据存放在内存中
