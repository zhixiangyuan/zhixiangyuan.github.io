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
draft: true
---

# 1 事务管理

```sql
-- 开始事务
START TRANSACTION;

-- 事务回滚
ROLLBACK;

-- 提交事务
COMMIT;
```

# 1.1 哪些语句可以被回滚

INSERT、UPDATE 和 DELETE 可以被回滚，SELECT 语句回滚没有用，CREATE、DROP 语句不会被回滚。

