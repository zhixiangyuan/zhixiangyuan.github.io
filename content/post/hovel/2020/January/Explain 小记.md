---
title: "Explain 小记"
date: 2020-01-18T16:44:45+08:00
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

explain 工具的作用是用于返回 sql 的执行计划，帮助开发人员优化 sql。

以下是本文使用的数据表的建表语句

```sql
DROP TABLE IF EXISTS `actor`;

CREATE TABLE `actor` (
  `id` int(11) AUTO_INCREMENT NOT NULL,
  `name` varchar(45) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

DROP TABLE IF EXISTS `film`;

CREATE TABLE `film` (
  `id` int(11) AUTO_INCREMENT NOT NULL,
  `name` varchar(100) NOT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_name` (`name`) 
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

DROP TABLE IF EXISTS `film_actor`;

CREATE TABLE `film_actor` (
  `id` int(11) AUTO_INCREMENT NOT NULL,
  `film_id` int(11) NOT NULL,
  `actor_id` int(11) NOT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_film_actor_id` (`film_id`,`actor_id`) 
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

# 1 使用方式

explain 用于和 select 配合使用，通过在 select 语句之前加上 explain 即可让 mysql 返回 sql 的执行计划而不是实际去执行获取执行结果。下面先简单来看一下 explain 的使用效果

```sql
EXPLAIN SELECT * FROM film;
```

![该图是 sql 的执行效果](/hub/2020/January/5.png)

可以看到 explain 的列有 id、select_type、table、type、possible_keys、key、key_len、ref、rows 和 Extra，下面我们就一一来分析每个列的含义。

## 1.1 id 列

id 列的编号是 select 的序列号，有几个 select 就有几个 id，并且 id 的顺序是按 select 出现的顺序增长的。id 越大执行优先级越高，id 相同则从上往下执行，id 为 null 最后执行。

下面是一个示例

```sql
EXPLAIN 
SELECT
    ( SELECT 1 FROM actor WHERE id = 1 ) 
FROM
    ( SELECT * FROM film WHERE id = 1 ) tmp;
```

![](/hub/2020/January/12.png)

## 1.2 select_type 列

select_type 列就像它名字所写的一样，表示的是查询的类型。查询的类型分为 simple、primary、subquery、derived、union 和 union result，下面就一一来看这几种查询的含义。

### 1.2.1 simple 类型

简单查询，查询不包含子查询和 union

下面是一个例子

```sql
EXPLAIN SELECT * FROM film;
```

![](/hub/2020/January/9.png)

需要注意，联表查询同样是 simple 类型

下面是一个例子

```sql
EXPLAIN SELECT * FROM film f LEFT JOIN film_actor fa ON fa.film_id = f.id;
```

![](/hub/2020/January/8.png)

### 1.2.2 primary 类型

复杂查询中最外层的查询，这里既然它是最外层查询，那么应该是最后执行的查询，那么 id 应该等于 1。

下面是一个例子

```sql
EXPLAIN 
SELECT
    ( SELECT 1 FROM actor WHERE id = 1 ) 
FROM
    ( SELECT * FROM film WHERE id = 1 ) tmp;
```

![](/hub/2020/January/10.png)

### 1.2.3 subquery 类型

subquery 表示包含在 select 与 from 之间的查询

下面是一个例子

```sql
EXPLAIN SELECT ( SELECT 1 FROM actor WHERE id = 1 ) FROM film;
```

![](/hub/2020/January/13.png)

### 1.2.4 derived 类型

该类型表示查询在 from 关键字后面

下面是一个例子

```sql
EXPLAIN SELECT 1 FROM ( SELECT * FROM film WHERE id = 1 ) tmp;
```

![](/hub/2020/January/11.png)

### 1.2.5 union 类型

union 类型表示 union 关键字后面的 select

下面是一个例子

```sql
EXPLAIN select 1 union select 1;
```

![](/hub/2020/January/14.png)

### 1.2.6 union result 类型

union result 类型的 id 为 null 表示其不是 select 操作，它是通过 union 来计算结果的操作

下面是一个例子

```sql
EXPLAIN select 1 union select 1;
```

![](/hub/2020/January/15.png)

## 1.3 table 列

table 表示该查询需要访问哪些表，一般都直接显示表名，但是有两种特殊情况，如果该表是 from 后面的 select 查出来的临时表，那么会写成 `<derived<$id>>` 的形式 derived 便是 from 后面的 select 的 select_type，`<$id>` 便是该查询的 id。还有一种特殊形式是有 union 时，UNION RESULT 的 table 列的值为 `<union <$id>,<$id>>` union 表示 union 操作，`<$id>` 同样表示 select 的 id。

下面的例子展示 table 为 `<derived<$id>>` 的情况

```sql
EXPLAIN SELECT 1 FROM ( SELECT * FROM film WHERE id = 1 ) tmp;
```

![](/hub/2020/January/11.png)

下面的例子展示 table 为 `<union <$id>,<$id>>` 的情况

![](/hub/2020/January/15.png)

## 1.4 type 列

这里暂时理解为有 system > const > eq_ref > ref > range > index > all 这些级别，一般查询需要优化到 range 级别。

null 表示不需要执行时访问表，可以直接找到该值

// todo 这里写的不够详细，对各种级别的含义需要做详细的阐述

## 1.5 possible_keys 列

该列显示查询时可能使用哪些索引来进行查询，显示时显示索引名称。如果该列为 null，那么可以考虑根据查询所用的字段加上索引。

## 1.6 key 列

该列显示实际 MySQL 采用哪个索引对该表进行查询。如果不使用索引进行查询，那么该列为显示为 null。

如果想强制 MySQL 使用或忽视 possible_keys 列中的索引，在查询中使用 force index 与 ignore index

以下是强制使用索引的案例

```sql
EXPLAIN SELECT * FROM film_actor FORCE INDEX(`idx_film_actor_id`) WHERE film_id = 1
```

![](/hub/2020/January/17.png)

以下是强制忽略索引的案例

```sql
EXPLAIN SELECT * FROM film_actor IGNORE INDEX(`idx_film_actor_id`) WHERE film_id = 1
```

![](/hub/2020/January/16.png)

如果 possible_keys 有索引，而 key 为 null 的情况，那么这种情况是因为表中数据不多，MySQL 认为索引对此查询帮助不大，于是选择了全表扫描。不过这种情况没有模拟出来，暂时没有案例可看了。

## 1.7 key_len 列

该列显示了 mysql 在索引里使用的字节数，通过这个值可以估算出具体使用了索引的哪些列。

估算时使用根据具体索引列使用的数据字节长度来估算

字符串：

1. char(n)：n 字节长度
2. varchar(n)：n 字节存储字符串长度，如果是 utf-8，则长度是 3n + 2

数值类型：

1. tinyint：1 字节
2. smallint：2 字节
3. int：4 字节
4. bigint：8 字节

时间类型：

1. date：3 字节
2. timestamp：4 字节
3. datetime：8 字节

下面是一个例子

film_actor 定义了 `KEY idx_film_actor_id (film_id,actor_id) ` 索引，film_id 和 actor_id 的数据类型都是 int

```sql
EXPLAIN SELECT * from film_actor where film_id =1;
```

可以看到，由于只使用了索引的第一列，所以 key_len 的长度是 4

![](/hub/2020/January/19.png)

```sql
EXPLAIN SELECT * FROM film_actor WHERE film_id = 1 AND actor_id = 1;
```

可以看到，由于只使用了索引的第一列和第二列，所以 key_len 的长度是 8

![](/hub/2020/January/18.png)


```sql
EXPLAIN SELECT * FROM film_actor WHERE actor_id = 1;
```

可以看到如果我们的 sql 中只使用了第二列在 where 条件中，mysql 依然会使用完全索引来进行查找

![](/hub/2020/January/20.png)

## 1.8 ref 列

// todo

## 1.9 rows 列

该列是 mysql 估计要读取并检测的行数，注意这个不是结果集的行数，只是估计的行数，并不准确。

## 1.10 Extra 列

一些额外信息

### 1.10.1 Using index

该信息表示此次查询使用了索引

```sql
EXPLAIN SELECT * FROM film_actor WHERE film_id = 1;
```

![](/hub/2020/January/19.png)

### 1.10.2 Using index condition

// todo


### 1.10.3 Using where

// todo

```sql
EXPLAIN SELECT * FROM `actor` WHERE `name` = 'a';
```

![](/hub/2020/January/21.png)

### 1.10.4 Using temporary

// todo

```sql
EXPLAIN SELECT DISTINCT `name` FROM `actor`;
```

![](/hub/2020/January/22.png)

### 1.10.5 Using filesort

// todo

```sql
EXPLAIN SELECT * FROM actor ORDER BY `name`;
```

![](/hub/2020/January/23.png)

### 1.10.6 Select tables optimized away

// todo

```sql
EXPLAIN SELECT min(id) FROM film;
```

![](/hub/2020/January/24.png)

# 参考文章

1. [Mysql 关键字 Explain 性能优化神器](https://aijishu.com/a/1060000000005994)