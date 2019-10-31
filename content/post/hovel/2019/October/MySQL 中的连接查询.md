---
title: "MySQL 中的连接查询"
date: 2019-10-30T14:02:29+08:00
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

# 1 交叉连接：CROSS JOIN

```shell
# 交叉连接使用这个数据表
mysql> SELECT * FROM orders;
+----+---------+
| id | cust_id |
+----+---------+
|  1 |       1 |
|  2 |    NULL |
+----+---------+
mysql> SELECT * FROM customers;
+----+-----------------+---------------+
| id | cust_name       | cust_contact  |
+----+-----------------+---------------+
|  1 | Albert Einstein | +1 3257184377 |
|  2 | NULL            | NULL          |
+----+-----------------+---------------+
```

交叉联结会选取指定的表的所有行，然后以笛卡尔积的形式组合

```shell
# 直接用 ',' 号隔开便是交叉连接
mysql>
SELECT
  c.id AS customers_id,
  c.cust_name AS customers_cust_name,
  o.id AS order_id,
  o.cust_id AS order_cust_id
FROM
  orders AS o, customers AS c;
+--------------+---------------------+----------+---------------+
| customers_id | customers_cust_name | order_id | order_cust_id |
+--------------+---------------------+----------+---------------+
|            1 | Albert Einstein     |        1 |             1 |
|            1 | Albert Einstein     |        2 |          NULL |
|            2 | NULL                |        1 |             1 |
|            2 | NULL                |        2 |          NULL |
+--------------+---------------------+----------+---------------+

# 同样可以使用关键字 CROSS JOIN
mysql>
SELECT
  c.id AS customers_id,
  c.cust_name AS customers_cust_name,
  o.id AS order_id,
  o.cust_id AS order_cust_id
FROM
  orders AS o CROSS JOIN customers AS c;
+--------------+---------------------+----------+---------------+
| customers_id | customers_cust_name | order_id | order_cust_id |
+--------------+---------------------+----------+---------------+
|            1 | Albert Einstein     |        1 |             1 |
|            1 | Albert Einstein     |        2 |          NULL |
|            2 | NULL                |        1 |             1 |
|            2 | NULL                |        2 |          NULL |
+--------------+---------------------+----------+---------------+
```

# 2 内连接：INNER JOIN 

```shell
# 内连接使用这个数据表
mysql> SELECT * FROM orders;
+----+---------+
| id | cust_id |
+----+---------+
|  1 |       1 |
|  2 |    NULL |
+----+---------+
mysql> SELECT * FROM customers;
+----+-----------------+---------------+
| id | cust_name       | cust_contact  |
+----+-----------------+---------------+
|  1 | Albert Einstein | +1 3257184377 |
|  2 | NULL            | NULL          |
+----+-----------------+---------------+
```

```shell
# 内连接查询在 MySQL 里面和交叉查询效果一样
mysql>
SELECT
  c.id AS customers_id,
  c.cust_name AS customers_cust_name,
  o.id AS order_id,
  o.cust_id AS order_cust_id
FROM
  orders AS o INNER JOIN customers AS c;
+--------------+---------------------+----------+---------------+
| customers_id | customers_cust_name | order_id | order_cust_id |
+--------------+---------------------+----------+---------------+
|            1 | Albert Einstein     |        1 |             1 |
|            1 | Albert Einstein     |        2 |          NULL |
|            2 | NULL                |        1 |             1 |
|            2 | NULL                |        2 |          NULL |
+--------------+---------------------+----------+---------------+

# 带连接条件的查询
mysql>
SELECT
  c.id AS customers_id,
  c.cust_name AS customers_cust_name,
  o.id AS order_id,
  o.cust_id AS order_cust_id
FROM
  orders AS o 
  INNER JOIN customers AS c ON o.cust_id = c.id;
+--------------+---------------------+----------+---------------+
| customers_id | customers_cust_name | order_id | order_cust_id |
+--------------+---------------------+----------+---------------+
|            1 | Albert Einstein     |        1 |             1 |
+--------------+---------------------+----------+---------------+

# 可以将条件放到 WHERE，但是 ON 可能能够提升性能
# 注意只是可能提升性能，没有测试过效果
mysql>
SELECT
  c.id AS customers_id,
  c.cust_name AS customers_cust_name,
  o.id AS order_id,
  o.cust_id AS order_cust_id
FROM
  orders AS o 
  INNER JOIN customers AS c
WHERE
  o.cust_id = c.id;
+--------------+---------------------+----------+---------------+
| customers_id | customers_cust_name | order_id | order_cust_id |
+--------------+---------------------+----------+---------------+
|            1 | Albert Einstein     |        1 |             1 |
+--------------+---------------------+----------+---------------+

# INNER JOIN 可以缩写为 JOIN，它们是等效的
mysql>
SELECT
  c.id AS customers_id,
  c.cust_name AS customers_cust_name,
  o.id AS order_id,
  o.cust_id AS order_cust_id
FROM
  orders AS o 
  JOIN customers AS c
WHERE
  o.cust_id = c.id;
+--------------+---------------------+----------+---------------+
| customers_id | customers_cust_name | order_id | order_cust_id |
+--------------+---------------------+----------+---------------+
|            1 | Albert Einstein     |        1 |             1 |
+--------------+---------------------+----------+---------------+
```

## 2.1 等值连接

```shell
# ON o.cust_id = c.id； ON 后面跟的条件是 = 的就是等值连接
mysql>
SELECT
  c.id AS customers_id,
  c.cust_name AS customers_cust_name,
  o.id AS order_id,
  o.cust_id AS order_cust_id
FROM
  orders AS o 
  INNER JOIN customers AS c ON o.cust_id = c.id;
+--------------+---------------------+----------+---------------+
| customers_id | customers_cust_name | order_id | order_cust_id |
+--------------+---------------------+----------+---------------+
|            1 | Albert Einstein     |        1 |             1 |
+--------------+---------------------+----------+---------------+
```

## 2.2 不等连接

```shell
# ON o.cust_id < c.id 这个条件不用 = 号则表示不等连接
mysql>
SELECT
  c.id AS customers_id,
  c.cust_name AS customers_cust_name,
  o.id AS order_id,
  o.cust_id AS order_cust_id
FROM
  orders AS o 
  INNER JOIN customers AS c ON o.cust_id < c.id;
+--------------+---------------------+----------+---------------+
| customers_id | customers_cust_name | order_id | order_cust_id |
+--------------+---------------------+----------+---------------+
|            2 | NULL                |        1 |             1 |
+--------------+---------------------+----------+---------------+
```

## 2.3 自连接

```shell
# 同一张表做连接就是自连接
SELECT
  *
FROM
  orders AS o1, orders AS o2;
+----+---------+----+---------+
| id | cust_id | id | cust_id |
+----+---------+----+---------+
|  1 |       1 |  1 |       1 |
|  2 |    NULL |  1 |       1 |
|  1 |       1 |  2 |    NULL |
|  2 |    NULL |  2 |    NULL |
+----+---------+----+---------+
```

# 3 外连接：OUTER JOIN

外连接分为两种 LEFT OUTER JOIN、RIGHT OUTER JOIN

```shell
# 外连接使用这个数据作为例子
mysql> SELECT * FROM orders;
+----+---------+
| id | cust_id |
+----+---------+
|  1 |       1 |
|  2 |    NULL |
+----+---------+
mysql> SELECT * FROM customers;
+----+-----------------+---------------+
| id | cust_name       | cust_contact  |
+----+-----------------+---------------+
|  1 | Albert Einstein | +1 3257184377 |
|  2 | NULL            | NULL          |
+----+-----------------+---------------+
```

## 3.1 左外连接：LEFT OUTER JOIN

效果看下面的例子，不好描述

```shell
SELECT
  *
FROM
  orders AS o 
  LEFT OUTER JOIN customers AS c ON o.cust_id = c.id;
+----+---------+------+-----------------+---------------+
| id | cust_id | id   | cust_name       | cust_contact  |
+----+---------+------+-----------------+---------------+
|  1 |       1 |    1 | Albert Einstein | +1 3257184377 |
|  2 |    NULL | NULL | NULL            | NULL          |
+----+---------+------+-----------------+---------------+
# LEFT OUTER JOIN 可以简写为 LEFT JOIN
```

## 3.2 右外连接：RIGHT OUTER JOIN

效果看下面的例子，不好描述

```shell
mysql>
SELECT
  *
FROM
  orders AS o 
  RIGHT OUTER JOIN customers AS c ON o.cust_id = c.id;
+------+---------+----+-----------------+---------------+
| id   | cust_id | id | cust_name       | cust_contact  |
+------+---------+----+-----------------+---------------+
|    1 |       1 |  1 | Albert Einstein | +1 3257184377 |
| NULL |    NULL |  2 | NULL            | NULL          |
+------+---------+----+-----------------+---------------+
# RIGHT OUTER JOIN 可以简写为 RIGHT JOIN
```

# 4 联合查询：UNION 和 UNION ALL

```shell
# 联合查询部分使用这个例子
mysql> SELECT * FROM customers;
+----+----------------------+---------------+
| id | cust_name            | cust_contact  |
+----+----------------------+---------------+
|  1 | Albert Einstein      | +1 3257184377 |
|  2 | Ludwig van Beethoven | +1 2014799186 |
|  3 | Sir Isaac Newton     | +1 5022799186 |
+----+----------------------+---------------+
```

# 4.1 UNION 的使用

```shell
# 先查一次
mysql> SELECT * FROM customers c WHERE c.id <= 2;
+----+----------------------+---------------+
| id | cust_name            | cust_contact  |
+----+----------------------+---------------+
|  1 | Albert Einstein      | +1 3257184377 |
|  2 | Ludwig van Beethoven | +1 2014799186 |
+----+----------------------+---------------+
# 再查一次
mysql> SELECT * FROM customers c WHERE c.id >= 2;
+----+----------------------+---------------+
| id | cust_name            | cust_contact  |
+----+----------------------+---------------+
|  2 | Ludwig van Beethoven | +1 2014799186 |
|  3 | Sir Isaac Newton     | +1 5022799186 |
+----+----------------------+---------------+
# 使用 UNION 组合结果，会起到去重的效果
mysql>
SELECT * FROM customers c WHERE c.id <= 2
UNION
SELECT * FROM customers c WHERE c.id >= 2;
+----+----------------------+---------------+
| id | cust_name            | cust_contact  |
+----+----------------------+---------------+
|  1 | Albert Einstein      | +1 3257184377 |
|  2 | Ludwig van Beethoven | +1 2014799186 |
|  3 | Sir Isaac Newton     | +1 5022799186 |
+----+----------------------+---------------+
```

# 4.1 UNION ALL 

```shell
# UNION ALL 不起去重的效果
mysql>
SELECT * FROM customers c WHERE c.id <= 2
UNION ALL
SELECT * FROM customers c WHERE c.id >= 2;
+----+----------------------+---------------+
| id | cust_name            | cust_contact  |
+----+----------------------+---------------+
|  1 | Albert Einstein      | +1 3257184377 |
|  2 | Ludwig van Beethoven | +1 2014799186 |
|  2 | Ludwig van Beethoven | +1 2014799186 |
|  3 | Sir Isaac Newton     | +1 5022799186 |
+----+----------------------+---------------+
```

# 5 全连接：FULL JOIN

MySQL 并不支持全连接，不过可以通过组合 RIGHT OUTER JOIN、LEFT OUTER JOIN 和 UNION 的方式实现全连接

```shell
# 先看一眼数据
mysql> SELECT * FROM orders;
+----+---------+
| id | cust_id |
+----+---------+
|  1 |       1 |
|  2 |    NULL |
+----+---------+
mysql> SELECT * FROM customers;
+----+-----------------+---------------+
| id | cust_name       | cust_contact  |
+----+-----------------+---------------+
|  1 | Albert Einstein | +1 3257184377 |
|  2 | NULL            | NULL          |
+----+-----------------+---------------+
# 使用全连接查询
SELECT
  *
FROM
  orders AS o 
  RIGHT OUTER JOIN customers AS c ON o.cust_id = c.id
UNION
SELECT
  *
FROM
  orders AS o 
  LEFT OUTER JOIN customers AS c ON o.cust_id = c.id;
+------+---------+------+-----------------+---------------+
| id   | cust_id | id   | cust_name       | cust_contact  |
+------+---------+------+-----------------+---------------+
|    1 |       1 |    1 | Albert Einstein | +1 3257184377 |
| NULL |    NULL |    2 | NULL            | NULL          |
|    2 |    NULL | NULL | NULL            | NULL          |
+------+---------+------+-----------------+---------------+
```

# 参考资料

1. [多表查询](https://www.zsythink.net/archives/1105)