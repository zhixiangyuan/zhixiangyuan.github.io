---
title: "MySQL 中的汇聚函数"
date: 2019-10-29T11:08:19+08:00
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

# 1 MySQL 中的汇聚函数

| 函数             | 说明             |
| ---------------- | ---------------- |
| `AVG(<field>)`   | 返回某列的平均值 |
| `COUNT(<field>)` | 返回某列的行数   |
| `MAX(<field>)`   | 返回某列的最大值 |
| `MIN(<field>)`   | 返回某列的最小值 |
| `SUM(<field>)`   | 返回某列值之和   |

以下表数据作为测试用例

```shell
mysql> SELECT * FROM _table;
+----+---------+------+---------------------+
| id | name    | age  | modify_date         |
+----+---------+------+---------------------+
|  1 |  Celia  |    1 | 2005-09-01 11:30:05 |
|  2 | Fraser  |    2 | 2005-09-01 00:00:00 |
|  3 | Pearl   |    3 | 2006-09-01 11:30:05 |
|  4 | Pamela  |    4 | 2012-08-01 11:30:05 |
|  5 | Marquis |    5 | 2013-10-01 11:30:05 |
|  6 | Ciaran  |    6 | 2014-11-01 11:30:05 |
|  7 | Y.Lee   | NULL | 2017-12-01 11:30:05 |
+----+---------+------+---------------------+
```

## 1.1 `AVG(<field>)`

```shell
# (1 + 2 + 3 +  4 + 5 + 6) / 6 = 3.5
# 注意：NULL 是不会参与计算的
mysql> SELECT AVG(age) FROM _table;
+----------+
| AVG(age) |
+----------+
|   3.5000 |
+----------+
```

## 1.2 `COUNT(<field>)`

```shell
mysql> SELECT COUNT(id) FROM _table;
+-----------+
| COUNT(id) |
+-----------+
|         7 |
+-----------+
# 注意：NULL 会被忽略
mysql> SELECT COUNT(age) FROM _table;
+------------+
| COUNT(age) |
+------------+
|          6 |
+------------+
```

## 1.3 `MAX(<field>)`

```shell
mysql> SELECT MAX(age) FROM _table;
+----------+
| MAX(age) |
+----------+
|        6 |
+----------+
```

## 1.4 `MIN(<field>)`

```shell
mysql> SELECT MIN(age) FROM _table;
+----------+
| MIN(age) |
+----------+
|        1 |
+----------+
```

## 1.5 `SUM(<field>)`

```shell
mysql> SELECT SUM(age) FROM _table;
+----------+
| SUM(age) |
+----------+
|       21 |
+----------+
```
