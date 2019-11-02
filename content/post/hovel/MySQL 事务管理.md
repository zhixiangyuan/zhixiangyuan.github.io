---
title: "MySQL 事务管理"
date: 2019-11-02T21:21:35+08:00
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

# 1 事务管理

以下的语句是连接隔离的，每个连接都有自己的环境。

```sql
-- 开始事务
START TRANSACTION;

-- 事务回滚
ROLLBACK;

-- 提交事务
COMMIT;

-- 设置保存点
SAVEPOINT <$savepoint_name>;

-- 回滚到保存点
-- 保存点在事务处理完成后会自动释放
-- 也就是执行 ROLLBACK 或 COMMIT 后自动释放
-- 这里回滚保存点有一点需要注意，它内部应该是一个栈的结构
-- 比如设置了 savepoint1、savepoint2、savepoint3 三个点，
-- 顺序也是按照 1、2、3 来的如果回滚到 2，那么就不能再回
-- 滚到 3 了，不过可以多次回滚到 2
ROLLBACK TO <$savepoint_name>;

-- 释放指定的保存点
RELEASE SAVEPOINT <$savepoint_name>;
```

默认情况下 sql 语句都是自动提交的，可以更改这个设置。

```sql
-- 改为不自动提交 sql
-- 这项修改是针对连接的，不是针对服务器的。
-- 改完之后如果不 COMMIT 那么语句就不生效
SET autocommit=0;
```

## 1.1 哪些语句可以被回滚

INSERT、UPDATE 和 DELETE 可以被回滚，SELECT 语句回滚没有用，CREATE、DROP 语句不会被回滚。

