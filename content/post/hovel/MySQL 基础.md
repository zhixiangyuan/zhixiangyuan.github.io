---
title: "MySQL 基础"
date: 2019-10-28T09:53:55+08:00
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

# 1 mysql 登陆命令

```shell
# 使用命令登陆 mysql
$> mysql -p <port> -h <host> -u <username> -p<password> 
```

# 2 一组 SHOW 命令

```shell
# 显示数据库列表
mysql> SHOW DATABASES;

# 选择某一个数据库
# 接下来输入的命令便是在该数据库中操作
mysql> USE <database name>

# 返回当前选择的数据库内的列表
mysql> SHOW TABLES;

# 显示某个表中列的相关信息
# 信息包括字段名、数据类型、是否允许为 NULL、键信息、默认值
# 以及扩展信息（如 auto_increment)。
mysql> SHOW COLUMNS FROM <table name>;
# 以下展示的便是输入该命令后返回的结果
+----------------+---------------------+------+-----+-------------------+-----------------------------+
| Field          | Type                | Null | Key | Default           | Extra                       |
+----------------+---------------------+------+-----+-------------------+-----------------------------+
| id             | bigint(20) unsigned | NO   | PRI | NULL              | auto_increment              |
| dailymotion_id | varchar(200)        | YES  |     | NULL              |                             |
| account        | varchar(30)         | YES  |     | NULL              |                             |
| password       | varchar(30)         | YES  |     | NULL              |                             |
| access_token   | varchar(2000)       | YES  |     | NULL              |                             |
| refresh_token  | varchar(2000)       | YES  |     | NULL              |                             |
| gmt_create     | datetime            | YES  |     | CURRENT_TIMESTAMP |                             |
| gmt_modified   | datetime            | YES  |     | CURRENT_TIMESTAMP | on update CURRENT_TIMESTAMP |
| is_delete      | tinyint(3) unsigned | YES  |     | 0                 |                             |
+----------------+---------------------+------+-----+-------------------+-----------------------------+

# 该语句等价于 SHOW COLUMNS FROM <table name>;
mysql> DESCRIBE <table name>;

# 显示服务器状态的信息
mysql> SHOW STATUS;
# 以下仅展示部分返回结果
+ ----------------------------------------------- +------------------------- +
| Variable_name                                   | Value                    |
+ ----------------------------------------------- + ------------------------ +
| Aborted_clients                                 | 0                        |
| Aborted_connects                                | 3                        |
| Binlog_cache_disk_use                           | 0                        |
| Binlog_cache_use                                | 0                        |
| Binlog_stmt_cache_disk_use                      | 0                        |
| Binlog_stmt_cache_use                           | 0                        |
+ ----------------------------------------------- + ------------------------ +

# 显示创建数据库的语句
mysql> SHOW CREATE DATABASE <database name>;
# 以下为返回结果
+----------+----------------------------------------------------------------------+
| Database | Create Database                                                      |
+----------+----------------------------------------------------------------------+
| cultuarl | CREATE DATABASE `cultuarl` /*!40100 DEFAULT CHARACTER SET utf8mb4 */ |
+----------+----------------------------------------------------------------------+

# 显示创建表的语句
mysql> SHOW CREATE TABLE <table name>;

# 用来显示授权用户的安全权限
mysql> SHOW GRANTS;
+---------------------------------------------------------------------+
| Grants for root@localhost                                           |
+---------------------------------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' WITH GRANT OPTION |
| GRANT PROXY ON ''@'' TO 'root'@'localhost' WITH GRANT OPTION        |
+---------------------------------------------------------------------+

# 显示服务器错误消息
mysql> SHOW ERRORS;
+-------+------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
| Level | Code | Message                                                                                                                                                |
+-------+------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
| Error | 1064 | You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'root' at line 1 |
+-------+------+--------------------------------------------------------------------------------------------------------------------------------------------------------+

# 显示服务器警告消息
mysql> SHOW WARNINGS;
# 奇怪的是展示警告消息会把错误消息也带出来
+-------+------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
| Level | Code | Message                                                                                                                                                |
+-------+------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
| Error | 1064 | You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'root' at line 1 |
+-------+------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
```

# 3 SELECT 命令浅析

```shell
# 检索单列数据
mysql> SELECT <field> FROM <table>;

# 检索多列数据，列与列之间用 ',' 隔开即可
# 注意：最后一个列名后面不需要加逗号
mysql> SELECT <field>,<field> FROM <table>;

# 检索所有列
# 注意：尽可能不要使用 '*' 否则由于查询字段过多很可能会影响性能
mysql> SELECT * FROM <table>;
```

## 3.1 DISTINCT 的使用

```shell
# 只有一个字段时比较好理解
mysql> SELECT DISTINCT <field> FROM <table>;
# 当有两个字段时效果如下所示
mysql> SELECT DISTINCT <field>,<field> FROM <table>;
# 下面是一个例子
    mysql> SELECT * FROM test_table;
    +------+-----------+
    | id   | second_id |
    +------+-----------+
    |    1 |         1 |
    |    2 |         2 |
    |    1 |         2 |
    |    2 |         1 |
    |    1 |         1 |
    +------+-----------+
    mysql> SELECT DISTINCT id,second_id from test_table;
    +------+-----------+
    | id   | second_id |
    +------+-----------+
    |    1 |         1 |
    |    2 |         2 |
    |    1 |         2 |
    |    2 |         1 |
    +------+-----------+
    # 可以发现，两个字段的时候 DISTINCT 的效果便是
    # 当两个字段组合起来是唯一标识的情况
# 需要注意的是 DISTINCT 必须前置于所有字段前面，像下面这样的语句会报错
# mysql>SELECT second_id,DISTINCT id from test_table
```

## 3.2 LIMIT 的使用

```shell
# 注意：MySQL 行号的第一行是 0
# 以下语句表示从第一行开始向下找几个记录，也就是 [1, 1 + <record number> - 1]
# 这里 1 + <record number> - 1 中 - 1 减掉的是第一行的那个元素
mysql> SELECT <field> FROM <table> LIMIT <record number>;
# 这种给参数的形式表示从 <start line number> 行开始，向下找 <record number> 个
# 也就是 [<start line number>, <start line number> + <record number> - 1]
mysql> SELECT <field> FROM <table> LIMIT <start line number>,<record number>;
# 从 MySQL 5 开始支持另一种写法，含义与上述语句相同
mysql> SELECT <field> FROM <table> LIMIT <record number> OFFSET <start line number>;

# 还有一种是完全限定的表名写法
mysql> SELECT <table>.<field> FROM <table>;
```

## 3.3 ORDER BY 的使用

```shell
# 按照选定的列来排序
mysql> SELECT <field> FROM <table> ORDER BY <field>;

# 选定多个列进行排序
# 当选定多个列来排序的时候，会先按照 field1 来排序，如果 field1 相同
# 则再按照 field2 来排序
mysql> SELECT <field> FROM <table> ORDER BY <field1>,<field2>;

# 选择排序方向升序或者降序
# 升序：从上到下越来越大，关键字 DESC
# 降序：从上到下越来越小，关键字 ASC
# 如果不选择的话，那么默认升序
mysql> SELECT <field> FROM <table> ORDER BY <field> [DESC|ASC]
# 当出现下述的语句时，升降序只对 field1 有效，field2 还是默认升序
mysql> SELECT <field> FROM <table> ORDER BY <field1> [DESC|ASC],<field2>;
```

## 3.4 LIMIT 和 ORDER BY 的组合使用

```shell
# 通过将它们组合使用，可以很轻松的找出最大值和最小值
# 以下语句可以找出最大值
mysql> SELECT <field> FROM <table> ORDER BY <field> DESC LIMIT 1;
# 以下语句可以找出最小值
mysql> SELECT <field> FROM <table> ORDER BY <field> ASC LIMIT 1;
```

## 3.5 WHERE 的使用

注意：当同时使用 WHERE 和 ORDER BY 的时候，ORDER BY 需要位于 WHERE 之后，否则会产生错误

```shell
# 只返回 <field> = <value> 的行
mysql> SELECT <field> FROM <table> WHERE <field> = <value>;
# 返回 <field> != <value> 的行
mysql> SELECT <field> FROM <table> WHERE <field> != <value>;
# 下面这句和上面等价
mysql> SELECT <field> FROM <table> WHERE <field> <> <value>;
# 查找 [<value1>, <value2>] 之间的值，注意是双边闭区间
mysql> SELECT <field> FROM <table> WHERE <field> BETWEEN <value1> AND <value2>;

# 字段为 NULL
mysql> SELECT <field> FROM <table> WHERE <field> IS NULL;
```

以下是可在 WHERE 中使用的操作符

| 操作符  | 说明               |
| ------- | ------------------ |
| =       | 等于               |
| <>      | 不等于             |
| !=      | 不等于             |
| <       | 小于               |
| <=      | 小于等于           |
| >       | 大于               |
| >=      | 大于等于           |
| BETWEEN | 在指定的两个值之间 |

## 3.6 在 WHERE 中使用 AND 与 OR

```shell
# AND 表示同时满足条件一和条件二，也就是交集
mysql> SELECT <field> FROM <table> WHERE <field1> = <value1> AND <field2> = <value2>;

# OR 表示满足条件一或者条件二，也就是并集
mysql> SELECT <field> FROM <table> WHERE <field1> = <value1> OR <field2> = <value2>;
```

注意：AND 的优先级比 OR 高，不过使用的时候还是加上括号比较好

## 3.7 在 WHERE 中使用 IN

```shell
# <field> IN (value1,value2) 等同于 <field> = <value1> OR <field> = <value2>
mysql> SELECT <field> FROM <table> WHERE <field> IN (value1,value2);
```

## 3.8 在 WHERE 中使用 NOT

```shell
# NOT 的作用是否定它后面所跟的条件
mysql> SELECT <field> FROM <table> WHERE <field> NOT IN (value1,value2);
# 上述语句等价于 
mysql> SELECT <field> FROM <table> WHERE <field> != <value1> OR <field> != <value2>;

mysql> SELECT <field> FROM <table> WHERE NOT <field> = <value>;
# 上述语句等价于
mysql> SELECT <field> FROM <table> WHERE <field> != <value>;
```

注意：NOT 的优先级低于 AND 和 OR

## 3.9 在 WHERE 中使用 LIKE

```shell
# % 表示任意字符出现任意次数
mysql> SELECT <field> FROM <table> WHERE <field> LIKE '<value>%';
# _ 表示任意字符出现一次，不能多也不能少
mysql> SELECT <field> FROM <table> WHERE <field> LIKE '<value>_';
```

### 3.9.1 使用通配符的注意事项

1. 如果其他操作符能达到相同的权限，应该使用其他操作符
2. 除非绝对必须，否则不要将通配符放到字符串开始处，否则搜索的很慢
3. 仔细注意通配符的位置，放错位置则不会返回想要的数据

## 3.10 在 WHERE 中使用正则表达式

```shell
# REGEXP 后面使用标准的正则表达式就行了
mysql> SELECT <field> FROM <table> WHERE <field> REGEXP '<regexp>';
```

## 3.11 使用 as 设置别名

```shell
# 使用 as 设置别名 <alias>
mysql> SELECT <field> as <alias> FROM <table>;
```

## 3.12 使用计算字段

```shell
# SELECT 后面的 field 可以是一个计算出来的字段，或者是函数处理出来的字段
mysql SELECT <field1>+<field2> FROM <table>;
mysql SELECT <field1>-<field2> FROM <table>;
mysql SELECT <field1>*<field2> FROM <table>;
mysql SELECT <field1>/<field2> FROM <table>;
# 支持加减乘除四种运算
```

# 4 什么时候使用引号 

使用 `''` 引起来的字段表示字符串，不引的一般是数值。比如后面这个 `'testStr'`


# 参考资料 

1. 《MySQL 必知必会》