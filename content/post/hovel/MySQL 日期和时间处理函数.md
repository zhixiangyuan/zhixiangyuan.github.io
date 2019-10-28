---
title: "MySQL 日期和时间处理函数"
date: 2019-10-28T20:37:26+08:00
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

# 1 日期和时间处理函数

| 函数                                        | 说明                                   |
| ------------------------------------------- | -------------------------------------- |
| `Date(<field>)`                             | 返回日期时间的日期部分                 |
| `Time(<field>)`                             | 返回一个日期时间的时间部分             |
| `Year(<field>)`                             | 返回一个日期的年份部分                 |
| `Month(<field>)`                            | 返回一个日期的月份部分                 |
| `Day(<field>)`                              | 返回一个日期的天数部分                 |
| `Hour(<field>)`                             | 返回一个时间的小时部分                 |
| `Minute(<field>)`                           | 返回一个时间的分钟部分                 |
| `Second(<field>)`                           | 返回一个时间的秒部分                   |
| `AddDate(<field>, INTERVAL <value> <unit>)` | 增加时间日期等，功能强大，可选时间单位 |
| `AddTime(<field>, <time>)`                  | 增加时间，单位默认为秒                 |
| `DateDiff(<date1>, <date2>)`                | 计算两个日期之差                       |
| `Date_Add(<date>, INTERVAL <value> <unit>)` | 高度灵活的日期运算函数                 |
| `Date_Format()`                             | 返回一个格式化的日期或时间串           |
| `DayOfWeek()`                               | 对于一个日期，返回对应的星期几         |
| `Now()`                                     | 返回当前日期和时间                     |
| `CurDate()`                                 | 返回当前日期                           |
| `CurTime()`                                 | 返回当前时间                           |


处理时间时使用 `yyyy-MM-dd HH:mm:ss` 这种时间格式，方式造成歧义

```shell
# 先看一下表中的数据，下面的例子均用这个表中的数据
mysql> SELECT * FROM _table;
```

## 1.1 `Date(<field>)`

```shell
# 如果直接使用这种比较方式，那么 MySQL 会把 time 当成 00:00:00 比较
mysql> SELECT * FROM _table WHERE modify_date = '2005-09-01';
+----+--------+---------------------+
| id | name   | modify_date         |
+----+--------+---------------------+
|  2 | Fraser | 2005-09-01 00:00:00 |
+----+--------+---------------------+

# 如果使用 DATE(modify_date) = '2005-09-01‘，那么就会忽略时间，仅比较日期
mysql> SELECT * FROM _table WHERE DATE(modify_date) = '2005-09-01';
+----+---------+---------------------+
| id | name    | modify_date         |
+----+---------+---------------------+
|  1 |  Celia  | 2005-09-01 11:30:05 |
|  2 | Fraser  | 2005-09-01 00:00:00 |
+----+---------+---------------------+

# DATE(modify_date) 还可以 BETWEEN 连用
mysql> SELECT * FROM _table WHERE DATE(modify_date) BETWEEN '2005-09-01' AND '2006-09-01';;
+----+---------+---------------------+
| id | name    | modify_date         |
+----+---------+---------------------+
|  1 |  Celia  | 2005-09-01 11:30:05 |
|  2 | Fraser  | 2005-09-01 00:00:00 |
|  3 | Pearl   | 2006-09-01 11:30:05 |
+----+---------+---------------------+
```

`Time(<field>)`、`Year(<field>)`、`Month(<field>)`、`Day(<field>)`、`Hour(<field>)`、`Minute(<field>)`、`Second(<field>)` 用法与 `Date(<field>)` 相同

## 1.2 `AddDate()`

语法：

- `ADDDATE(<field>, <days>)`
- `ADDDATE(<field>, INTERVAL <value> <time unit>)`

### 1.2.2 `ADDDATE(<field>, <days>)`

这种属于帮字段 field 增加 days 天

```shell
# 下面这个例子就是 modify_date 增加了一天
mysql> SELECT id, name, AddDate(modify_date, 1) FROM _table;
+----+---------+-------------------------+
| id | name    | AddDate(modify_date, 1) |
+----+---------+-------------------------+
|  1 |  Celia  | 2005-09-02 11:30:05     |
|  2 | Fraser  | 2005-09-02 00:00:00     |
|  3 | Pearl   | 2006-09-02 11:30:05     |
|  4 | Pamela  | 2012-08-02 11:30:05     |
|  5 | Marquis | 2013-10-02 11:30:05     |
|  6 | Ciaran  | 2014-11-02 11:30:05     |
|  7 | Y.Lee   | 2017-12-02 11:30:05     |
+----+---------+-------------------------+
```

### 1.2.1 `ADDDATE(<field>, INTERVAL <value> <unit>)`

这种语法自由一点，可以选择时间单位，time unit 有以下的选择

- MICROSECOND
- SECOND
- MINUTE
- HOUR
- DAY
- WEEK
- MONTH
- QUARTER
- YEAR
- SECOND_MICROSECOND
- MINUTE_MICROSECOND
- MINUTE_SECOND
- HOUR_MICROSECOND
- HOUR_SECOND
- HOUR_MINUTE
- DAY_MICROSECOND
- DAY_SECOND
- DAY_MINUTE
- DAY_HOUR
- YEAR_MONTH

时间单位比较多，根据情况选择

```shell
# 这个语句就等同于  AddDate(modify_date, 1)
mysql> SELECT id, name, AddDate(modify_date, INTERVAL 1 DAY) FROM _table;
+----+---------+--------------------------------------+
| id | name    | AddDate(modify_date, INTERVAL 1 DAY) |
+----+---------+--------------------------------------+
|  1 |  Celia  | 2005-09-02 11:30:05                  |
|  2 | Fraser  | 2005-09-02 00:00:00                  |
|  3 | Pearl   | 2006-09-02 11:30:05                  |
|  4 | Pamela  | 2012-08-02 11:30:05                  |
|  5 | Marquis | 2013-10-02 11:30:05                  |
|  6 | Ciaran  | 2014-11-02 11:30:05                  |
|  7 | Y.Lee   | 2017-12-02 11:30:05                  |
+----+---------+--------------------------------------+
# 这样能起到加一秒的效果
mysql> SELECT id, name, AddDate(modify_date, INTERVAL 1 SECOND) FROM _table;
+----+---------+-----------------------------------------+
| id | name    | AddDate(modify_date, INTERVAL 1 SECOND) |
+----+---------+-----------------------------------------+
|  1 |  Celia  | 2005-09-01 11:30:06                     |
|  2 | Fraser  | 2005-09-01 00:00:01                     |
|  3 | Pearl   | 2006-09-01 11:30:06                     |
|  4 | Pamela  | 2012-08-01 11:30:06                     |
|  5 | Marquis | 2013-10-01 11:30:06                     |
|  6 | Ciaran  | 2014-11-01 11:30:06                     |
|  7 | Y.Lee   | 2017-12-01 11:30:06                     |
+----+---------+-----------------------------------------+
# 这样和 AddDate(modify_date, INTERVAL 1 SECOND) 没区别
mysql> SELECT id, name, AddDate(modify_date, INTERVAL 1 HOUR_SECOND) FROM _table;
+----+---------+---------------------------------------------+
| id | name    | AddDate(modify_date, INTERVAL 1 DAY_SECOND) |
+----+---------+---------------------------------------------+
|  1 |  Celia  | 2005-09-01 11:30:06                         |
|  2 | Fraser  | 2005-09-01 00:00:01                         |
|  3 | Pearl   | 2006-09-01 11:30:06                         |
|  4 | Pamela  | 2012-08-01 11:30:06                         |
|  5 | Marquis | 2013-10-01 11:30:06                         |
|  6 | Ciaran  | 2014-11-01 11:30:06                         |
|  7 | Y.Lee   | 2017-12-01 11:30:06                         |
+----+---------+---------------------------------------------+
# 这样和 AddDate(modify_date, INTERVAL 1 HOUR_SECOND) 好像是一样的
mysql> SELECT id, name, AddDate(modify_date, INTERVAL 1 DAY_SECOND) FROM _table;
+----+---------+----------------------------------------------+
| id | name    | AddDate(modify_date, INTERVAL 1 HOUR_SECOND) |
+----+---------+----------------------------------------------+
|  1 |  Celia  | 2005-09-01 11:30:06                          |
|  2 | Fraser  | 2005-09-01 00:00:01                          |
|  3 | Pearl   | 2006-09-01 11:30:06                          |
|  4 | Pamela  | 2012-08-01 11:30:06                          |
|  5 | Marquis | 2013-10-01 11:30:06                          |
|  6 | Ciaran  | 2014-11-01 11:30:06                          |
|  7 | Y.Lee   | 2017-12-01 11:30:06                          |
+----+---------+----------------------------------------------+
# 所以 SECOND、HOUR_SECOND、DAY_SECOND 是一样的
```

## 1.3 `AddTime(<field>, <time>)`

这里默认单位是秒

```shell
# 秒钟加一
mysql> SELECT id, name, AddTime(modify_date, 1) FROM _table;
+----+---------+-------------------------+
| id | name    | AddTime(modify_date, 1) |
+----+---------+-------------------------+
|  1 |  Celia  | 2005-09-01 11:30:06     |
|  2 | Fraser  | 2005-09-01 00:00:01     |
|  3 | Pearl   | 2006-09-01 11:30:06     |
|  4 | Pamela  | 2012-08-01 11:30:06     |
|  5 | Marquis | 2013-10-01 11:30:06     |
|  6 | Ciaran  | 2014-11-01 11:30:06     |
|  7 | Y.Lee   | 2017-12-01 11:30:06     |
+----+---------+-------------------------+
# 还可以带点小数
mysql> SELECT id, name, AddTime(modify_date, 1.001) FROM _table;
+----+---------+-----------------------------+
| id | name    | AddTime(modify_date, 1.001) |
+----+---------+-----------------------------+
|  1 |  Celia  | 2005-09-01 11:30:06.001     |
|  2 | Fraser  | 2005-09-01 00:00:01.001     |
|  3 | Pearl   | 2006-09-01 11:30:06.001     |
|  4 | Pamela  | 2012-08-01 11:30:06.001     |
|  5 | Marquis | 2013-10-01 11:30:06.001     |
|  6 | Ciaran  | 2014-11-01 11:30:06.001     |
|  7 | Y.Lee   | 2017-12-01 11:30:06.001     |
+----+---------+-----------------------------+
```

## 1.4 `DateDiff(<date1>, <date2>)`

```shell
# 前面的时间要大于后面的时间计算出来才是正的，而且计算出来的时间天数是
# [<date1>, <date2>) 这样一个区间，时间单位是天
mysql> SELECT DATEDIFF("2017-01-01", "2017-01-02");
+--------------------------------------+
| DATEDIFF("2017-01-01", "2017-01-02") |
+--------------------------------------+
|                                   -1 |
+--------------------------------------+
mysql> SELECT DATEDIFF("2017-01-02", "2017-01-01");
+--------------------------------------+
| DATEDIFF("2017-01-02", "2017-01-01") |
+--------------------------------------+
|                                    1 |
+--------------------------------------+
```

## 1.5 `Date_Add(<date>, INTERVAL <value> <unit>)`

这里就不多说了，时间单位和 `ADDDATE(<field>, INTERVAL <value> <unit>)` 一样

```shell
mysql> SELECT DATE_ADD("2017-06-15", INTERVAL 10 DAY);
+-----------------------------------------+
| DATE_ADD("2017-06-15", INTERVAL 10 DAY) |
+-----------------------------------------+
| 2017-06-25                              |
+-----------------------------------------+
```

## 1.6 `Date_Format(<date>, <format>)`

format 多种格式可选，同时可以使用组合格式

| Format | Description                                                                  |
| ------ | ---------------------------------------------------------------------------- |
| %a     | Abbreviated weekday name (Sun to Sat)                                        |
| %b     | Abbreviated month name (Jan to Dec)                                          |
| %c     | Numeric month name (0 to 12)                                                 |
| %D     | Day of the month as a numeric value, followed by suffix (1st, 2nd, 3rd, ...) |
| %d     | Day of the month as a numeric value (01 to 31)                               |
| %e     | Day of the month as a numeric value (0 to 31)                                |
| %f     | Microseconds (000000 to 999999)                                              |
| %H     | Hour (00 to 23)                                                              |
| %h     | Hour (00 to 12)                                                              |
| %I     | Hour (00 to 12)                                                              |
| %i     | Minutes (00 to 59)                                                           |
| %j     | Day of the year (001 to 366)                                                 |
| %k     | Hour (0 to 23)                                                               |
| %l     | Hour (1 to 12)                                                               |
| %M     | Month name in full (January to December)                                     |
| %m     | Month name as a numeric value (00 to 12)                                     |
| %p     | AM or PM                                                                     |
| %r     | Time in 12 hour AM or PM format (hh:mm:ss AM/PM)                             |
| %S     | Seconds (00 to 59)                                                           |
| %s     | Seconds (00 to 59)                                                           |
| %T     | Time in 24 hour format (hh:mm:ss)                                            |
| %U     | Week where Sunday is the first day of the week (00 to 53)                    |
| %u     | Week where Monday is the first day of the week (00 to 53)                    |
| %V     | Week where Sunday is the first day of the week (01 to 53). Used with %X      |
| %v     | Week where Monday is the first day of the week (01 to 53). Used with %X      |
| %W     | Weekday name in full (Sunday to Saturday)                                    |
| %w     | Day of the week where Sunday=0 and Saturday=6                                |
| %X     | Year for the week where Sunday is the first day of the week. Used with %V    |
| %x     | Year for the week where Monday is the first day of the week. Used with %V    |
| %Y     | Year as a numeric, 4-digit value                                             |
| %y     | Year as a numeric, 2-digit value                                             |

```shell
# 几个例子
mysql> SELECT DATE_FORMAT("2017-06-15", "%Y");
+---------------------------------+
| DATE_FORMAT("2017-06-15", "%Y") |
+---------------------------------+
| 2017                            |
+---------------------------------+
mysql> SELECT DATE_FORMAT("2017-06-15", "%d/%m/%Y");
+---------------------------------------+
| DATE_FORMAT("2017-06-15", "%d/%m/%Y") |
+---------------------------------------+
| 15/06/2017                            |
+---------------------------------------+
mysql> SELECT DATE_FORMAT("2017-06-15", "%d/%M/%Y");
+---------------------------------------+
| DATE_FORMAT("2017-06-15", "%d/%M/%Y") |
+---------------------------------------+
| 15/June/2017                          |
+---------------------------------------+
```

## 1.7 `DayOfWeek(<date>)`

下表为数字与 week 的对应关系，注意 1 代表周天

| result | week      |
| ------ | --------- |
| 1      | Sunday    |
| 2      | Monday    |
| 3      | Tuesday   |
| 4      | Wednesday |
| 5      | Thursday  |
| 6      | Friday    |
| 7      | Saturday  |

```shell
mysql> SELECT DayOfWeek('2019-10-28');
+-------------------------+
| DayOfWeek('2019-10-28') |
+-------------------------+
|                       2 |
+-------------------------+
```

## 1.8 `Now()` 

```shell
mysql> SELECT Now();
+---------------------+
| Now()               |
+---------------------+
| 2019-10-28 14:15:38 |
+---------------------+
```

## 1.9  `CurDate()`

```shell
mysql> SELECT CurDate();
+------------+
| CurDate()  |
+------------+
| 2019-10-28 |
+------------+
```

## 1.10 `CurTime()`

```shell
mysql> SELECT CurTime();
+-----------+
| CurTime() |
+-----------+
| 14:16:38  |
+-----------+
```

这里记录的也不全，更全的参考见参考资料

# 参考资料

1. [SQL Tutorial](https://www.w3schools.com/sql/func_mysql_adddate.asp)