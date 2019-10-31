---
title: "MySQL UPDATE、DELETE 与 TRUNCATE 的使用"
date: 2019-10-31T10:39:22+08:00
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

# 1 UPDATE 语句的使用

注意：UPDATE 语句的使用一定要小心，因为一不小心你可能就更新了表中的所有数据。

```shell
# 先看一下表结构
mysql> SELECT * FROM prods;
+----+-----------+-------------+
| id | prod_name | prod_number |
+----+-----------+-------------+
|  1 | apple     |           1 |
|  2 | banana    |           2 |
+----+-----------+-------------+

# 更新 banana 为 mango，库存改为 100
mysql>
UPDATE prods
SET prod_name = 'mango', prod_number = '100'
WHERE id = 2;

# 看看是否修改完成
mysql> SELECT * FROM prods;
+----+-----------+-------------+
| id | prod_name | prod_number |
+----+-----------+-------------+
|  1 | apple     |           1 |
|  2 | mango     |         100 |
+----+-----------+-------------+

# 将某个字段置为 NULL，这里的 NULL 不是字符串，是真的 NULL
mysql>
UPDATE prods
SET prod_name = NULL, prod_number = '100'
WHERE id = 2;

# 查看一下置为 NULL 的效果
mysql> SELECT * FROM prods;
+----+-----------+-------------+
| id | prod_name | prod_number |
+----+-----------+-------------+
|  1 | apple     |           1 |
|  2 | NULL      |         100 |
+----+-----------+-------------+

# 添加 LOW_PRIORITY 关键字使查询优先
mysql>
UPDATE LOW_PRIORITY prods
SET prod_name = 'mango', prod_number = '100'
WHERE id = 2;

# 此时输入会报错的语句
mysql> UPDATE prods SET id = 1 WHERE id = 2;
ERROR 1062 (23000): Duplicate entry '1' for key 'PRIMARY'

# 通过增加 IGNORE 忽略报错
mysql> UPDATE IGNORE prods SET id = 1 WHERE id = 2;
Query OK, 0 rows affected, 1 warning (0.00 sec)
Rows matched: 1  Changed: 0  Warnings: 1
```

# 2 DELETE 语句的使用

注意：DELETE 语句使用时一定要小心，因为一不小心你可能就删除了所有的数据。

```shell
# 先看一下表结构
mysql> SELECT * FROM prods;
+----+-----------+-------------+
| id | prod_name | prod_number |
+----+-----------+-------------+
|  1 | apple     |           1 |
|  2 | mango     |         100 |
+----+-----------+-------------+

# 删除表中的数据
mysql> DELETE FROM prods WHERE id = 2;
Query OK, 1 row affected (0.00 sec)

# 删除不存在的数据也不会报错
mysql> DELETE FROM prods WHERE id = 2;
Query OK, 0 rows affected (0.00 sec)

# 删除表中的所有数据
mysql> DELETE FROM prods;
```

# 3 TRUNCATE 语句的使用

```shell
# 删除表中的所有数据时可以用这个语句
# 作用时删除原来的表，然后重新创建一个
mysql> TRUNCATE <table_name>;
```

# 参考资料 

1. 《MySQL 必知必会》