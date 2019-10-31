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
-- [IF NOT EXISTS] 如果不加这个，那么如果表已存在的报错
CREATE TABLE [IF NOT EXISTS] <table_name> (
    -- <field_name> 字段名
    -- <data_type> 数据类型
    -- [size] todo 貌似是字段长度，具体作用待定
    -- [NOT NULL|NULL] 是否可以为 NULL
    -- [DEFAULT <value>] 是否有默认值
    -- [AUTO_INCREMENT] 是否自增，每张表只能有一个自增字段，并且必须给这个字段加索引，比如主键
    <field_name> <data_type>[size] [NOT NULL|NULL] [DEFAULT <value>] [AUTO_INCREMENT],
    [field_name data_type]...
    -- 设置主键
    [PRIMARY KEY (<field>, [field...])]
)
-- ENGING 选择引擎
ENGING=InnoDB;

CREATE TABLE [IF NOT EXISTS] <table_name> (
    <field_name> <data_type>[size] [NOT NULL|NULL] [DEFAULT <value>] [AUTO_INCREMENT],
    [field_name data_type]...
    [PRIMARY KEY (<field>, [field...])]
)ENGING=InnoDB;

CREATE TABLE IF NOT EXISTS test (
    id  int
);
```


# 参考资料

1. [MySQL创建表](https://www.yiibai.com/mysql/create-table.html)
2. [MYSQL 中的 COLLATE 是什么？](https://juejin.im/post/5bfe5cc36fb9a04a082161c2)