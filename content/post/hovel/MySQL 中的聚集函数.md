---
title: "MySQL 中的聚集函数"
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

# 1 MySQL 中的聚集函数

| 函数                               | 说明             |
| ---------------------------------- | ---------------- |
| `AVG([DISTINCT or ALL] <field>)`   | 返回某列的平均值 |
| `COUNT([DISTINCT or ALL] <field>)` | 返回某列的行数   |
| `MAX(<field>)`                     | 返回某列的最大值 |
| `MIN(<field>)`                     | 返回某列的最小值 |
| `SUM([DISTINCT or ALL] <field>)`   | 返回某列值之和   |

- DISTINCT 参数起到去重的作用
- ALL 为默认参数，不指定则默认使用所有值

以下表数据作为测试用例

```shell
mysql> SELECT * FROM _table;
+----+---------+------+---------------------+
| id | name    | age  | modify_date         |
+----+---------+------+---------------------+
|  1 |         |    1 | 2005-09-01 11:30:05 |
|  2 | Fraser  |    2 | 2005-09-01 00:00:00 |
|  3 | Pearl   |    3 | 2006-09-01 11:30:05 |
|  4 | Pamela  |    4 | 2012-08-01 11:30:05 |
|  5 | Marquis |    5 | 2013-10-01 11:30:05 |
|  6 | Ciaran  |    6 | 2014-11-01 11:30:05 |
|  7 |         |    6 | 2017-12-01 11:30:05 |
|  8 | NULL    | NULL | NULL                |
+----+---------+------+---------------------+
```

## 1.1 `AVG(<field>)`

```shell
# (1 + 2 + 3 +  4 + 5 + 6 + 6) / 7 = 3.8571
# 注意：NULL 是不会参与计算的
mysql> SELECT AVG(age) FROM _table;
+----------+
| AVG(age) |
+----------+
|   3.8571 |
+----------+
# 使用 DISTINCT 去重
#  (1 + 2 + 3 +  4 + 5 + 6) / 6 = 3.5
mysql> SELECT AVG(DISTINCT age) FROM _table;
+-------------------+
| AVG(DISTINCT age) |
+-------------------+
|            3.5000 |
+-------------------+
```

## 1.2 `COUNT(<field>)`

```shell
mysql> SELECT COUNT(id) FROM _table;
+-----------+
| COUNT(id) |
+-----------+
|         8 |
+-----------+
# 注意：NULL 会被忽略
mysql> SELECT COUNT(age) FROM _table;
+------------+
| COUNT(age) |
+------------+
|          7 |
+------------+
# 使用 DISTINCT 去重
mysql> SELECT COUNT(DISTINCT age) FROM _table;
+---------------------+
| COUNT(DISTINCT age) |
+---------------------+
|                   6 |
+---------------------+
# 对所有行进行计数，不忽略 NULL
# 此时不能使用 DISTINCT
mysql> SELECT COUNT(*) FROM _table;
+----------+
| COUNT(*) |
+----------+
|        8 |
+----------+
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

注意：这个函数不要往文本上用，因为我测试过后返回的结果没什么规律

## 1.4 `MIN(<field>)`

```shell
mysql> SELECT MIN(age) FROM _table;
+----------+
| MIN(age) |
+----------+
|        1 |
+----------+
```

注意：这个函数不要往文本上用，因为我测试过后返回的结果没什么规律

## 1.5 `SUM(<field>)`

```shell
mysql> SELECT SUM(age) FROM _table;
+----------+
| SUM(age) |
+----------+
|       27 |
+----------+
```

# 2 组合聚集函数

```shell
# 多个聚集函数可以一起使用
mysql> SELECT COUNT(*) as line_num, MAX(age), MIN(age)  FROM _table;
+----------+----------+----------+
| line_num | MAX(age) | MIN(age) |
+----------+----------+----------+
|        8 |        6 |        1 |
+----------+----------+----------+
```

# 参考资料

1. 《MySQL 必知必会》