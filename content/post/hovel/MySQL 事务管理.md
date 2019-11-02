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
ROLLBACK TO <$savepoint_name>;

RELEASE
```

默认情况下 sql 语句都是自动提交的，可以更改这个设置。

```sql
-- 改为不自动提交 sql
-- 这项修改是针对连接的，不是针对服务器的。
SET autocommit=0;
```

## 1.1 哪些语句可以被回滚

INSERT、UPDATE 和 DELETE 可以被回滚，SELECT 语句回滚没有用，CREATE、DROP 语句不会被回滚。

