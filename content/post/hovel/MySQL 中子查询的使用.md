---
title: "MySQL 中子查询的使用"
date: 2019-10-30T10:43:43+08:00
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

# 1 测试用例使用的数据表

```shell
# 客户表，存储客户的信息
mysql> SELECT * FROM customers;
+----+----------------------+---------------+
| id | cust_name            | cust_contact  |
+----+----------------------+---------------+
|  1 | Albert Einstein      | +1 3257184377 |
|  2 | Ludwig van Beethoven | +1 2014799186 |
+----+----------------------+---------------+
# 订单表，存储客户订单的信息
mysql> SELECT * FROM orders;
+----+---------+
| id | cust_id |
+----+---------+
|  1 |       1 |
|  2 |       2 |
|  3 |       1 |
|  4 |       1 |
|  5 |       2 |
+----+---------+
# 商品表
mysql> SELECT * FROM prods;
+----+------------+
| id | prod_name  |
+----+------------+
|  1 | apple      |
|  2 | banana     |
|  3 | strawberry |
+----+------------+
# 订单和商品的关联表
mysql> SELECT * FROM orderitems;
+----+----------+---------+
| id | order_id | prod_id |
+----+----------+---------+
|  1 |        1 |       1 |
|  2 |        1 |       2 |
|  3 |        2 |       1 |
|  4 |        2 |       3 |
|  5 |        3 |       1 |
|  6 |        3 |       2 |
|  7 |        4 |       2 |
|  8 |        4 |       1 |
|  9 |        5 |       2 |
+----+----------+---------+
```

# 2 子查询用在 SELECT 后面

## 2.1 查看每个用户有多少订单

```shell
mysql> 
# 以下是均格式化 SQL 来看
SELECT
  c.id,
  c.cust_name,
  ( 
    SELECT 
      COUNT(*) 
    FROM 
      orders o 
    WHERE 
      o.cust_id = c.id 
  ) 
  AS order_num 
FROM
  customers c;
+----+----------------------+-----------+
| id | cust_name            | order_num |
+----+----------------------+-----------+
|  1 | Albert Einstein      |         3 |
|  2 | Ludwig van Beethoven |         2 |
+----+----------------------+-----------+
```

## 2.2 查看每个用户买过多少商品

```shell
mysql> 
SELECT
  c.id,
  c.cust_name,
  ( 
    SELECT 
      COUNT(oi.id) 
    FROM 
      orderitems oi
    LEFT JOIN
      orders o ON oi.order_id = o.id
    WHERE 
      o.cust_id = c.id 
  ) 
  AS order_num 
FROM
  customers c;
+----+----------------------+-----------+
| id | cust_name            | order_num |
+----+----------------------+-----------+
|  1 | Albert Einstein      |         6 |
|  2 | Ludwig van Beethoven |         3 |
+----+----------------------+-----------+

# 需要注意的是子查询返回的行数与列数需要与父查询需要的行数与列数对应
# 比如说下面的语句就会报错行数不对应
mysql>
SELECT
  c.id,
  c.cust_name,
  ( 
    SELECT 
      oi.id 
    FROM 
      orderitems oi
    LEFT JOIN
      orders o ON oi.order_id = o.id
    WHERE 
      o.cust_id = c.id 
  ) 
  AS order_num 
FROM
  customers c;
ERROR 1242 (21000): Subquery returns more than 1 row
# 以下 sql 会报列数不对应
mysql>
SELECT
  c.id,
  c.cust_name,
  ( 
    SELECT 
      oi.*
    FROM 
      orderitems oi
    LEFT JOIN
      orders o ON oi.order_id = o.id
    WHERE 
      o.cust_id = c.id 
  ) 
  AS order_num 
FROM
  customers c;
ERROR 1241 (21000): Operand should contain 1 column(s)
```

# 3 子查询用在 WHERE 后面

## 3.1 买过商品 apple 的订单

```shell
mysql>
# 子查询与 IN 连用的例子
SELECT 
  DISTINCT oi.order_id
FROM
  orderitems oi
WHERE
  oi.prod_id IN (
      SELECT
        prod_id
      FROM
        prods
      WHERE
        prod_name = 'apple'
  );
+----------+
| order_id |
+----------+
|        1 |
|        2 |
|        3 |
|        4 |
|        5 |
+----------+
```

## 3.2 买过商品 apple 的订单，并且是名叫 Albert Einstein 的用户下的订单

```shell
mysql>
# 子查询与 IN 和 = 连用的例子
SELECT 
  DISTINCT oi.order_id
FROM
  orderitems oi
LEFT JOIN
  orders o ON o.id = oi.order_id
WHERE
  oi.prod_id IN (
      SELECT
        prod_id
      FROM
        prods
      WHERE
        prod_name = 'apple'
  )
AND
  o.cust_id = (
      SELECT
        c.id
      FROM
        customers c
      WHERE
        c.cust_name = 'Albert Einstein'
  );
+----------+
| order_id |
+----------+
|        1 |
|        3 |
|        4 |
+----------+
```

# 参考资料

1. 《MySQL 必知必会》

# 附录

这里放的是建表语句

```sql
/*
 Navicat Premium Data Transfer

 Source Server         : github-mall
 Source Server Type    : MySQL
 Source Server Version : 50728
 Source Host           : localhost:3306
 Source Schema         : test

 Target Server Type    : MySQL
 Target Server Version : 50728
 File Encoding         : 65001

 Date: 30/10/2019 11:42:10
*/

SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for customers
-- ----------------------------
DROP TABLE IF EXISTS `customers`;
CREATE TABLE `customers` (
  `id` int(11) NOT NULL,
  `cust_name` varchar(255) COLLATE utf8mb4_bin DEFAULT NULL,
  `cust_contact` varchar(255) COLLATE utf8mb4_bin DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;

-- ----------------------------
-- Records of customers
-- ----------------------------
BEGIN;
INSERT INTO `customers` VALUES (1, 'Albert Einstein', '+1 3257184377');
INSERT INTO `customers` VALUES (2, 'Ludwig van Beethoven', '+1 2014799186');
COMMIT;

-- ----------------------------
-- Table structure for orderitems
-- ----------------------------
DROP TABLE IF EXISTS `orderitems`;
CREATE TABLE `orderitems` (
  `id` int(11) NOT NULL,
  `order_id` int(11) DEFAULT NULL,
  `prod_id` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;

-- ----------------------------
-- Records of orderitems
-- ----------------------------
BEGIN;
INSERT INTO `orderitems` VALUES (1, 1, 1);
INSERT INTO `orderitems` VALUES (2, 1, 2);
INSERT INTO `orderitems` VALUES (3, 2, 1);
INSERT INTO `orderitems` VALUES (4, 2, 3);
INSERT INTO `orderitems` VALUES (5, 3, 1);
INSERT INTO `orderitems` VALUES (6, 3, 2);
INSERT INTO `orderitems` VALUES (7, 4, 2);
INSERT INTO `orderitems` VALUES (8, 4, 1);
INSERT INTO `orderitems` VALUES (9, 5, 2);
COMMIT;

-- ----------------------------
-- Table structure for orders
-- ----------------------------
DROP TABLE IF EXISTS `orders`;
CREATE TABLE `orders` (
  `id` int(11) NOT NULL,
  `cust_id` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;

-- ----------------------------
-- Records of orders
-- ----------------------------
BEGIN;
INSERT INTO `orders` VALUES (1, 1);
INSERT INTO `orders` VALUES (2, 2);
INSERT INTO `orders` VALUES (3, 1);
INSERT INTO `orders` VALUES (4, 1);
INSERT INTO `orders` VALUES (5, 2);
COMMIT;

-- ----------------------------
-- Table structure for prods
-- ----------------------------
DROP TABLE IF EXISTS `prods`;
CREATE TABLE `prods` (
  `id` int(11) NOT NULL,
  `prod_name` varchar(255) COLLATE utf8mb4_bin DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;

-- ----------------------------
-- Records of prods
-- ----------------------------
BEGIN;
INSERT INTO `prods` VALUES (1, 'apple');
INSERT INTO `prods` VALUES (2, 'banana');
INSERT INTO `prods` VALUES (3, 'strawberry');
COMMIT;

SET FOREIGN_KEY_CHECKS = 1;
```