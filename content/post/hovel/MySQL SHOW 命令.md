---
title: "MySQL 控制以及查看状态的命令"
date: 2019-10-28T18:19:32+08:00
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

# 1 一组 SHOW 命令

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

# 参考资料

1. 《MySQL 必知必会》