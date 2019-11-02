---
title: "MySQL 中的用户和权限管理"
date: 2019-11-02T23:55:47+08:00
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

# 1 用户管理

MySql 用户账号存储在名为 mysql 的 MySQL 数据库中，一般不需要直接访问 mysql 数据库和表，通过相关命令即可完成操作。

```sql
-- 选择 mysql 数据库
USE mysql;
-- 查看所有用户
SELECT user FROM user;

-- 创建新用户
-- $password 在 mysql 中存储时是经过加密的
CREATE USER <$usernamee> IDENTIFIED BY '<$password>';

-- 用户重命名
-- 将 $username_old 用户名改成 $username_new
RENAME USER <$username_old> TO <$username_new>;

-- 删除用户和账号相关的权限
-- 5.0 之前的版本这个命令只能删除用户，不删除权限
DROP USER <$username>;

-- 修改指定用户密码
SET PASSWORD FOR <$username> = Password('<password_new>');

-- 修改本账户的密码
SET PASSWORD = Password('<password_new>');
```

# 2 权限管理

```sql
-- 查看用户权限
SHOW GRANTS FOR <$username>;

-- 授予 $permission 权限在 $database_name 库的 $table_name 表上
-- 给 $username 用户
GRANT <$permission>[,...] ON <$database_name>.<$table_name> TO <$username>;

-- 撤销权限，如果该权限不存在则报错
REVOKE <$permission> ON <$database_name>.<$table_name> FROM <$username>;
```

下面是权限表

| $permission             | 说明                                                                                    |
| ----------------------- | --------------------------------------------------------------------------------------- |
| ALL                     | 除 GRANT OPTION 外的所有权限                                                            |
| ALTER                   | 使用 ALTER TABLE                                                                        |
| ALTER ROUTINE           | 使用 ALTER PROCEDURE 和 DROP PROCEDURE                                                  |
| CREATE                  | 使用 CREATE TABLE                                                                       |
| CREATE ROUTINE          | 使用 CREATE PROCEDURE                                                                   |
| CREATE TEMPORARY TABLES | 使用 CREATE TEMPORARY TABLE                                                             |
| CREATE USER             | 使用 CREATE USER、DROP USER、RENAME USER 和 REVOKE ALL PRIVILEGES                       |
| CREATE VIEW             | 使用 CREATE VIEW                                                                        |
| DELETE                  | 使用 DELETE                                                                             |
| DROP                    | 使用 DROP TABLE                                                                         |
| EXECUTE                 | 使用 CALL 和存储过程                                                                    |
| FILE                    | 使用 SELECT INTO OUTFILE 和 LOAD DATA INFILE                                            |
| GRANT OPTION            | 使用 GRANT 和 REVOKE                                                                    |
| INDEX                   | 使用 CREATE INDEX 和 DROP INDEX                                                         |
| INSERT                  | 使用 INSERT                                                                             |
| LOCK TABLES             | 使用 LOCK TABLES                                                                        |
| PROCESS                 | 使用 SHOW FULL PROCESSLIST                                                              |
| RELOAD                  | 使用 FLUSH                                                                              |
| REPLICATION CLIENT      | 服务器位置的访问                                                                        |
| REPLICATION SLAVE       | 由复制从属使用                                                                          |
| SELECT                  | 使用 SELECT                                                                             |
| SHOW DATABASES          | 使用 SHOW DATABASES                                                                     |
| SHOW VIEW               | 使用 SHOW CREATE VIEW                                                                   |
| SHUTDOWN                | 使用 mysqladmin shutdown（用来关闭MySQL）                                               |
| SUPER                   | 使用 CHANGE MASTER、KILL、LOGS、PURGE、MASTER 和 SET GLOBAL。还允许 mysqladmin 调试登录 |
| UPDATE                  | 使用 UPDATE                                                                             |
| USAGE                   | 无访问权限                                                                              |

以下是一个例子

```shell
# peter 是我新创建的一个用户
# GRANT USAGE ON *.* TO 'peter'@'%'
# 这个权限等于没有权限
mysql> SHOW GRANTS FOR peter;
+-----------------------------------+
| Grants for peter@%                |
+-----------------------------------+
| GRANT USAGE ON *.* TO 'peter'@'%' |
+-----------------------------------+
# 授予 test 库的所有表的 SELECT 权限给 peter
mysql> GRANT SELECT ON test.* TO peter;
# 再看一下权限
mysql> SHOW GRANTS FOR peter;
+-----------------------------------------+
| Grants for peter@%                      |
+-----------------------------------------+
| GRANT USAGE ON *.* TO 'peter'@'%'       |
| GRANT SELECT ON `test`.* TO 'peter'@'%' |
+-----------------------------------------+
# 撤销 peter 的权限
mysql> REVOKE SELECT ON test.* FROM peter;
# 再看一下权限
mysql> SHOW GRANTS FOR peter;
+-----------------------------------+
| Grants for peter@%                |
+-----------------------------------+
| GRANT USAGE ON *.* TO 'peter'@'%' |
+-----------------------------------+
```