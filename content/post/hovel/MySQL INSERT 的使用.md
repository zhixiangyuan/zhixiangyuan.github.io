---
title: "MySQL INSERT 的使用"
date: 2019-10-31T10:05:58+08:00
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

# 1 插入单条数据

```shell
# 查看表结构
mysql> SELECT * FROM prods;
+----+------------+
| id | prod_name  |
+----+------------+
|  1 | apple      |
|  2 | banana     |
|  3 | strawberry |
+----+------------+

# 向表中插入数据
mysql> INSERT INTO prods(id, prod_name) VALUES(4, 'mango');
Query OK, 1 row affected (0.01 sec)

# 再次查看表结构
mysql>  SELECT * FROM prods;
+----+------------+
| id | prod_name  |
+----+------------+
|  1 | apple      |
|  2 | banana     |
|  3 | strawberry |
|  4 | mango      |
+----+------------+
# 如果主键可以自增，可以省略主键，让其自动生成，语句如下
mysql> INSERT INTO prods(prod_name) VALUES('mango');
```

# 2 插入多条数据

```shell
# 查看表结构
mysql>  SELECT * FROM prods;
+----+------------+
| id | prod_name  |
+----+------------+
|  1 | apple      |
|  2 | banana     |
|  3 | strawberry |
|  4 | mango      |
+----+------------+

# 可以插入的数值写多个就可以了
# 这样写要比使用多条插入语句效率高
mysql> INSERT INTO prods(id, prod_name) VALUES(5, 'pear'), (6, 'kiwi berry');
Query OK, 2 rows affected (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 0

# 再次查看表结构
mysql> SELECT * FROM prods;
+----+------------+
| id | prod_name  |
+----+------------+
|  1 | apple      |
|  2 | banana     |
|  3 | strawberry |
|  4 | mango      |
|  5 | pear       |
|  6 | kiwi berry |
+----+------------+

# 可以在 INSERT 和 INTO 关键字之间加 LOW_PRIORITY 降低插
# 入的优先级，这样做主要是为了保证查询的性能
mysql> INSERT LOW_PRIORITY INTO prods(id, prod_name) VALUES(7, 'orange');
Query OK, 1 row affected (0.00 sec)

# 查看添加进去的橘子
mysql> SELECT * FROM prods;
+----+------------+
| id | prod_name  |
+----+------------+
|  1 | apple      |
|  2 | banana     |
|  3 | strawberry |
|  4 | mango      |
|  5 | pear       |
|  6 | kiwi berry |
|  7 | orange     |
+----+------------+
```

# 3 插入查询出的数据

```shell
# 先看一下数据库里的数据
mysql> SELECT * FROM prods;
+----+-----------+
| id | prod_name |
+----+-----------+
|  1 | apple     |
|  2 | banana    |
+----+-----------+

# 插入查询出的数据
mysql> INSERT INTO prods(prod_name) SELECT prod_name FROM prods;
Query OK, 2 rows affected (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 0

# 可以看到插入的效果
mysql> SELECT * FROM prods;
+----+-----------+
| id | prod_name |
+----+-----------+
|  1 | apple     |
|  2 | banana    |
|  3 | apple     |
|  4 | banana    |
+----+-----------+
```

# 参考资料

1. 《MySQL 必知必会》