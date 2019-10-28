---
title: "MySQL 中常见函数使用示例"
date: 2019-10-28T15:32:01+08:00
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

# 1 在查询时实现字符串拼接

```shell
# 使用 Concat 函数
mysql> SELECT Concat(<field1>, [<field>...]) FROM vendors;

# 下面是一个例子
mysql> SELECT Concat(vend_name, '(', vend_country, ')') FROM vendors;
+------------------------------------------+
|Concat(vend_name, ' (', vend_country, ')')|
+------------------------------------------+
| ACME (USA)                               |
| Anvils R Us (USA)                        |
+------------------------------------------+

# 下面是另一个例子
mysql> SELECT * FROM test_table;
+------+--------+----------+
| id   | two_id | three_id |
+------+--------+----------+
|    1 |      1 |        1 |
|    2 |      2 |        2 |
|    1 |      2 |        3 |
|    2 |      1 |        4 |
|    1 |      1 |        5 |
|    3 |   NULL |        6 |
|    4 |   NULL |        7 |
+------+--------+----------+
mysql> SELECT Concat(id, '-', two_id, '-', three_id) FROM test_table;
+------------------------------------+
| Concat(id,'-',two_id,'-',three_id) |
+------------------------------------+
| 1-1-1                              |
| 2-2-2                              |
| 1-2-3                              |
| 2-1-4                              |
| 1-1-5                              |
| NULL                               |
| NULL                               |
+------------------------------------+
# 可以看到，如果拼接的字符串中有一个是 NULL，那么拼接出来的字符串就是 NULL
```

# 2 文本处理函数

| 函数                                    | 说明                     |
| --------------------------------------- | ------------------------ |
| `Left(<field>, <length>)`               | 返回字符串左边的字符     |
| `Right(<field>, <length>)`              | 返回字符串右边的字符     |
| `Length(<field>)`                       | 返回字符串的长度         |
| `Locate(<subStr>, <field>)`             | 找出字符串的一个子串     |
| `Lower(<field>)`                        | 将字符串转换为小写       |
| `Upper(<field>)`                        | 将字符串转换为大写       |
| `LTrim(<field>)`                        | 去掉字符串左边的空格     |
| `RTrim(<field>)`                        | 去掉字符串右边的空格     |
| `Trim(<field>)`                         | 去掉字符串左右两边的空格 |
| `Soundex(<field>)`                      | 返回字符串的 Soundex 值  |
| `SubString(<field>, <start>, <length>)` | 返回字符串的字符         |

注意：MySQL 里面的字符串位置是从 1 开始的。

以下是测试用例

```shell
# 先看以下表中所有的数据
mysql> SELECT * FROM _table;
+----+---------+
| id | name    |
+----+---------+
|  1 |  Celia  |
|  2 | Fraser  |
|  3 | Pearl   |
|  4 | Pamela  |
|  5 | Marquis |
|  6 | Ciaran  |
|  7 | Y.Lee   |
+----+---------+

# 试一下 Left(name, 1)，下标从 1 开始，找一个字母，也就是 [1, 1]
# 这里可以看到左边第一个是空格
mysql> SELECT id, Left(name, 1) FROM _table;
+----+---------------+
| id | Left(name, 1) |
+----+---------------+
|  1 |               |
|  2 | F             |
|  3 | P             |
|  4 | P             |
|  5 | M             |
|  6 | C             |
|  7 | Y             |
+----+---------------+

# 试一下 Right(name, 1)，下标从 1 开始，右边开始找一个字母，也就是 [length, length]
# 这里可以看到右边第一个是空格 
mysql> SELECT id, Right(name, 1) FROM _table;
+----+----------------+
| id | Right(name, 1) |
+----+----------------+
|  1 |                |
|  2 | r              |
|  3 | l              |
|  4 | a              |
|  5 | s              |
|  6 | n              |
|  7 | e              |
+----+----------------+

# 试一下 Length(name)
mysql> SELECT id, Length(name) FROM _table;
+----+--------------+
| id | Length(name) |
+----+--------------+
|  1 |            7 |
|  2 |            6 |
|  3 |            5 |
|  4 |            6 |
|  5 |            7 |
|  6 |            6 |
|  7 |            5 |
+----+--------------+

# 试一下 Locate('P', name)
mysql> SELECT id, Locate('P', name) FROM _table;
+----+-------------------+
| id | Locate('P', name) |
+----+-------------------+
|  1 |                 0 |
|  2 |                 0 |
|  3 |                 1 |
|  4 |                 1 |
|  5 |                 0 |
|  6 |                 0 |
|  7 |                 0 |
+----+-------------------+

# 试一下 Lower(name)
mysql> SELECT id, Lower(name) FROM _table;
+----+-------------+
| id | Lower(name) |
+----+-------------+
|  1 |  celia      |
|  2 | fraser      |
|  3 | pearl       |
|  4 | pamela      |
|  5 | marquis     |
|  6 | ciaran      |
|  7 | y.lee       |
+----+-------------+

# 试一下 Upper(name)
mysql> SELECT id, Upper(name) FROM _table;
+----+-------------+
| id | Upper(name) |
+----+-------------+
|  1 |  CELIA      |
|  2 | FRASER      |
|  3 | PEARL       |
|  4 | PAMELA      |
|  5 | MARQUIS     |
|  6 | CIARAN      |
|  7 | Y.LEE       |
+----+-------------+

# 试一下 LTrim(name)，可以看到左边的空格没有了
mysql> SELECT id, LTrim(name) FROM _table;
+----+-------------+
| id | LTrim(name) |
+----+-------------+
|  1 | Celia       |
|  2 | Fraser      |
|  3 | Pearl       |
|  4 | Pamela      |
|  5 | Marquis     |
|  6 | Ciaran      |
|  7 | Y.Lee       |
+----+-------------+

# 试一下 RTrim(name)，右边的空格其实去掉了
mysql> SELECT id, RTrim(name) FROM _table;
+----+-------------+
| id | RTrim(name) |
+----+-------------+
|  1 |  Celia      |
|  2 | Fraser      |
|  3 | Pearl       |
|  4 | Pamela      |
|  5 | Marquis     |
|  6 | Ciaran      |
|  7 | Y.Lee       |
+----+-------------+

# 试一下 Trim(name)，两边的空格都去掉了
mysql> SELECT id, Trim(name) FROM _table;
+----+------------+
| id | Trim(name) |
+----+------------+
|  1 | Celia      |
|  2 | Fraser     |
|  3 | Pearl      |
|  4 | Pamela     |
|  5 | Marquis    |
|  6 | Ciaran     |
|  7 | Y.Lee      |
+----+------------+

# 试一下 Soundex(name) = Soundex("Y.Lie")
# Soundex 其实是根据发音找的，Y.Lee 和 Y.Lie 发音相近，所以能查询到
mysql> SELECT id, name FROM _table WHERE Soundex(name) = Soundex("Y.Lie");
+----+-------+
| id | name  |
+----+-------+
|  7 | Y.Lee |
+----+-------+

# 试一下 SubString(name, 1, 1)，索引下标从 1 开始，然后找一个字符
# 所以就和 Left(name, 1) 效果相同
mysql> SELECT id, SubString(name, 1, 1) FROM _table;
+----+-----------------------+
| id | SubString(name, 1, 1) |
+----+-----------------------+
|  1 |                       |
|  2 | F                     |
|  3 | P                     |
|  4 | P                     |
|  5 | M                     |
|  6 | C                     |
|  7 | Y                     |
+----+-----------------------+

# 与 Concat() 组合使用
mysql> SELECT Concat(Trim(id), '-', Trim(name)) FROM _table;
+-----------------------------------+
| Concat(Trim(id), '-', Trim(name)) |
+-----------------------------------+
| 1-Celia                           |
| 2-Fraser                          |
| 3-Pearl                           |
| 4-Pamela                          |
| 5-Marquis                         |
| 6-Ciaran                          |
| 7-Y.Lee                           |
+-----------------------------------+
```

