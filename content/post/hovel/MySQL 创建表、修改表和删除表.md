---
title: "MySQL 创建表、修改表和删除表"
date: 2019-10-31T15:22:49+08:00
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

# 1 创建表

```sql

CREATE TABLE [IF NOT EXISTS] <table_name> (
    <field_name> <data_type>[size] [NOT NULL|NULL] [DEFAULT <value>] [AUTO_INCREMENT] [COMMENT <comment>] [COLLATE <collate>] [UNSIGNED],
    [field_name data_type]...
    [PRIMARY KEY (<field>, [field...])]
)[ENGING=<enging>] [CHARSET=<charset>] [COLLATE=<collate>];

-- 下面对各个选项打上注释
-- [IF NOT EXISTS] 如果不加这个，那么如果表已存在的报错
CREATE TABLE [IF NOT EXISTS] <table_name> (
    -- <field_name> 字段名
    <field_name> 
    -- <data_type> 数据类型
     -- [size] todo 貌似是字段长度，具体作用待定
    <data_type>[size] 
    -- [NOT NULL|NULL] 是否可以为 NULL，默认可以为 NULL
    [NOT NULL|NULL] 
    -- [DEFAULT <value>] 是否有默认值
    [DEFAULT <value>] 
    -- [AUTO_INCREMENT] 是否自增，每张表只能有一个自增字段，并且必须给这个字段加索引，比如主键
    [AUTO_INCREMENT] 
    -- [COMMENT <comment>] 注释字段
    [COMMENT <comment>] 
    -- [COLLATE <charset>] 解释在下面
    [COLLATE <collate>]
    -- [UNSIGNED] 对于数字类型表示其没有符号
    [UNSIGNED],
    [field_name data_type]...
    -- [PRIMARY KEY (<field>, [field...])] 设置主键
    [PRIMARY KEY (<field>, [field...])]
)
-- [ENGING=<enging>] 选择引擎，不选择则是默认引擎
[ENGING=<enging>]
-- [CHARSET=<charset>] 选择字符集
[CHARSET=<charset>] 
-- [COLLATE=<collate>] 选择排序规则
[COLLATE=<collate>];

CREATE TABLE test3 (
    id varchar(32) COLLATE utf8_bin,
    test int UNSIGNED
)CHARSET=utf8mb4 COLLATE=utf8mb4_bin;
```

# 1.1 [COLLATE <charset>] 的作用

COLLATE 用于指定排序规则，只有数据类型是 VARCHAR、CHAR、TEXT 类型的字段需要指定这个规则。这个排序规则会影响到 ORDER BY 和大小于号的排序，同时它对于索引也有一定的影响。

不同 CHARSET 都有一种默认的 COLLATE，例如 Latin1 编码的默认 COLLATE 为 latin1_swedish_ci，GBK 编码的默认 COLLATE 为 gbk_chinese_ci，utf8mb4 编码的默认值为 utf8mb4_general_ci。

很多 COLLATE 都带有 _ci 字样，这是 Case Insensitive 的缩写，即大小写无关。与此同时，对于那些 _cs 后缀的 COLLATE，则是 Case Sensitive，即大小写敏感的。

同时在 MySQL 中 utf8 编码是 utf8mb4， utf8 是一个历史遗留问题，它只支持 3 bytes，utf8mb4 支持 4 bytes，所以使用 utf8mb4 就好了。

# 参考资料

1. [MySQL创建表](https://www.yiibai.com/mysql/create-table.html)
2. [MYSQL 中的 COLLATE 是什么？](https://juejin.im/post/5bfe5cc36fb9a04a082161c2)