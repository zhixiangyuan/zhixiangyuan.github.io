---
title: "SQL 优化小记"
date: 2020-01-19T17:02:13+08:00
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

本篇文章使用的建表语句以及其创建数据的语句

```sql
DROP TABLE IF EXISTS employees;

CREATE TABLE `employees` (
	`id` INT ( 11 ) NOT NULL AUTO_INCREMENT,
	`name` VARCHAR ( 24 ) NOT NULL DEFAULT '' COMMENT '姓名',
	`age` INT ( 20 ) NOT NULL DEFAULT '0' COMMENT '年龄',
	`position` VARCHAR ( 20 ) NOT NULL DEFAULT '' COMMENT '职位',
	`hire_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '入职时间',
	PRIMARY KEY ( `id` ),
	KEY `idx_name_age_position` ( `name`, `age`, `position` ) USING BTREE 
) ENGINE = INNODB DEFAULT CHARSET = utf8 COMMENT = '员工表';

INSERT INTO employees(name,age,position,hire_time) VALUES('LiLei', 22, 'manager', NOW());
INSERT INTO employees(name,age,position,hire_time) VALUES('HanMeimei', 23, 'dev', NOW());
INSERT INTO employees(name,age,position,hire_time) VALUES('Lucy', 23, 'dev', NOW());

-- 这个语句用于刷数据，每次数据量翻一倍
INSERT INTO employees(name,age,position,hire_time) SELECT name,age,position,hire_time FROM employees;
```

# 1 最左前缀法则

对于多列索引，查询时不能跳跃省略左侧索引列的条件，否则不走索引，对于本文中的 `idx_name_age_position` 索引，我们看下面几个例子

```sql
EXPLAIN SELECT * FROM employees WHERE name = 'LiLei' AND age = 22;
```

可以看到，当左侧查询条件包含 name 和 age 时，实际查询时使用索引。

![](/hub/2020/January/27.png)

在看下面这个例子

```sql
EXPLAIN SELECT * FROM employees WHERE age = 22;
```

可以看到，我们跳过了 name，只使用 age 去查询，这个时候组合索引失效了。

![](/hub/2020/January/26.png)

这种类型的查询我们需要考虑为 age 字段单独加上索引，比如我们使用下面的 sql 加上索引。

```sql
-- 加上索引
ALTER TABLE `employees` ADD INDEX `idx_age` ( `age` ) USING BTREE;
-- 再次查询
EXPLAIN SELECT * FROM employees WHERE age = 22;
```

可以看到，加上索引之后的查询便通过索引了

![](/hub/2020/January/25.png)

我们用下面的 sql 将新加的索引去掉，还原表

```sql
ALTER TABLE `employees` DROP INDEX `idx_age`;
```

下面我们演示另一种组合索引部分失效的情况

```sql 
-- 先用这句 sql 看一下该组合索引的总长度
EXPLAIN SELECT * FROM employees WHERE name ='LiLei' AND age = 22 AND position = 'manager';
```

可以看到，该组合索引的总长度是 140，之前我们有试过只使用 name 和 age 时的索引长度，是 78

![](/hub/2020/January/33.png)

然后如果我们在查询的时候给 age 加上范围，会导致部分索引失效

```sql
EXPLAIN SELECT * FROM employees WHERE name ='LiLei' AND age > 22 AND position = 'manager';
```

可以看到，虽然我们的查询语句中使用到了 position，但其实只使用了索引前两列进行查询。

![](/hub/2020/January/34.png)

这里目前只知道这样写会出现索引部分失效，至于优化方案暂时不清楚。

# 2 索引失效

索引失效的情况很多，这里暂列几种

## 2.1 因对列进行计算

这里先看一种正常走索引的情况

```sql
EXPLAIN SELECT * FROM employees WHERE name = 'LiLei';
```

![](/hub/2020/January/28.png)

如果此时我们对 name 列进行计算，那么便不会再走索引

```sql
EXPLAIN SELECT * FROM employees WHERE LEFT(name, 3) = 'LiL';
```

![](/hub/2020/January/29.png)

对于这种形式的 sql 我们可以改写成下面这种形式，效果相同，但是会走索引

```sql
EXPLAIN SELECT * FROM employees WHERE name LIKE 'LiL%';
```

![](/hub/2020/January/30.png)

下面的 sql 也不会走索引

```sql
EXPLAIN SELECT * FROM employees WHERE id - 1 = 2;
```

这里索引失效同样是对左边的列进行计算导致的，我们只需要将表达式改写成 `id = 3` 便会走索引

![](/hub/2020/January/31.png)

下面是改写后的效果

```sql
EXPLAIN SELECT * FROM employees WHERE id = 3;
```

![](/hub/2020/January/32.png)

## 2.2 LIKE 使用不当

当使用 LIKE 时，如果将 % 放到最左边，那么同样不会走索引而是使用全表扫描

```sql
EXPLAIN SELECT * FROM employees WHERE name LIKE '%LiLei' ;
```

![](/hub/2020/January/38.png)

将 % 放右边就行了

```sql
EXPLAIN SELECT * FROM employees WHERE name LIKE 'LiL%';
```

![](/hub/2020/January/30.png)

对于这种问题的优化方式，目前的想法是直接和产品说无法开发此功能，路堵死，不给他搞。

# 3 尽量使用覆盖索引进行查询

覆盖索引的意思就是 SELECT 出的数据列最好只包含索引列，这里我们举个例子，假设又一个表 t(a,b,c,d,e,f)，其中 a 列是主键索引，b 列是辅助索引，那么此时磁盘上有两棵 b+ 树，即聚集索引和辅助索引，分别保存 (a,b,c,d,e,f) 和 (b,a)，如果查询条件中的 where 条件可以通过 b 列的索引过滤掉一部分记录，查询就会走辅助索引，如果用户只 SELECT a,b 的数据则直接通过辅助索引就可以知道用户查询的数据，如果是 SELECT *，那么当获取的数据不足时，还需要根据 a 去聚集索引上查找数据，那么此时还需要读取磁盘，那么性能会下降非常多。而且多查出来的数据经过网络传输也会消耗更多的时间，影响性能。

对于我们的这个表，下面的第一条查询语句会比第二条快很多

```sql
-- 第一条查询语句没有 hire_time，hire_time 上没有索引
SELECT id, name, age, position FROM employees WHERE name ='LiLei' AND age = 22 AND position = 'manager';

-- 第二条查询语句包含 hire_time，hire_time 上没有索引
SELECT id, name, age, position, hire_time FROM employees WHERE name ='LiLei' AND age = 22 AND position = 'manager';
```

下面我们实验一下效果，先来看一下 employees 这张表的记录总数

```sql
-- 先统计一下表中的数据量，需要注意的是，这里使用 COUNT(*)，
-- 而不要使用 COUNT(1) 或是 COUNT(<$字段>)，mysql 对于 COUNT(*)
-- 有特殊优化，速度会更快
SELECT COUNT(*) FROM employees;
```

可以看到这张表有 100W 的数据量，下面我们通过两个 sql 来查询一下，看看查询时间

![](/hub/2020/January/35.png)

我们下面先测试一下第一条 sql 语句的查询速度

![](/hub/2020/January/36.png)

然后来测试一下第二条 sql 的速度

![](/hub/2020/January/37.png)

从结果来看，第二条比第一条快了接近一倍

# 4 对于 ORDER BY 的优化

下面的语句跳过了 age 字段，造成没有利用索引做排序

```sql
EXPLAIN SELECT * FROM employees WHERE name = 'LiLei' ORDER BY position;
```

可以看到在 Extra 中有 Using filesort，说明做了额外的排序动作，浪费性能

![](/hub/2020/January/39.png)

下面我们补上 age 字段

```sql
EXPLAIN SELECT * FROM employees WHERE name = 'LiLei' AND age = '18' ORDER BY position;
```

![](/hub/2020/January/40.png)

可以看到，当利用了索引做排序之后便不再有 Using filesort，不过这里的 age 加上后改变了 sql 的语义，实际场景读者根据场景随机应变。

# 5 优化分页查询

对于下面这条 sql，实际执行的时候是不走索引的，执行时会先读取 100000 条数据，然后抛弃后面的 999995 条数据，最后剩下的便是需要查询的那 5 条数据，但是这样效率就很低。

```sql
EXPLAIN SELECT * FROM employees LIMIT 999995, 5;
```

![](/hub/2020/January/41.png)

下面是实际的执行效果，耗时 0.702 秒

![](/hub/2020/January/42.png)

对这条 sql 我们可以做如下的优化，让它走索引，提高查询效率

```sql
EXPLAIN SELECT * FROM employees WHERE id > 999995 LIMIT 5;
```

![](/hub/2020/January/43.png)

下面是实际的执行效果，0.003 秒，比不走索引快得多

![](/hub/2020/January/44.png)

但是这种优化方式不实用，实际场景的时候经常会删数据，导致主键不连续，那么这种方法就不能用了，因为查出来的数据不一样了。

# 6 优化非主键字段排序的分页查询

可以看到，以 name 排序的分页查询不走索引

```sql
EXPLAIN SELECT * FROM employees ORDER BY name LIMIT 999999, 5;
```

![](/hub/2020/January/45.png)

以下是实际的执行时间，可怕的 59 秒

![](/hub/2020/January/46.png)

这里我们可以优化为先经过条件查询出主键，然后根据主键查询需要的数据，并且这里的排序通过主键排序，没有 Using filesort 了

```sql
-- 优化后的 sql
EXPLAIN SELECT * FROM employees AS e 
INNER JOIN ( SELECT id FROM employees ORDER BY `name` LIMIT 999999, 5 ) AS ed ON e.id = ed.id; 
```

![](/hub/2020/January/47.png)

以下是执行耗时，0.471 秒

![](/hub/2020/January/48.png)

# 参考文章

1. [工作中遇到的 99% SQL 优化，这里都能给你解决方案](https://aijishu.com/a/1060000000008206)
2. [SQL 语句为什么使用 select * 会降低查询速度？](https://www.zhihu.com/question/37777220)
3. [工作中遇到的 99% SQL 优化，这里都能给你解决方案 (二)](https://aijishu.com/a/1060000000009624)
4. [工作中遇到的 99% SQL 优化，这里都能给你解决方案 (三)](https://aijishu.com/a/1060000000009631)