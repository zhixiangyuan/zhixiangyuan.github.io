---
title: "MySQL 视图的创建和删除"
date: 2019-11-01T16:45:48+08:00
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

视图便是临时表，在每次查看视图的时候执行指定的 SQL 语句。

# 1 视图的创建

```sql
-- 创建视图
CREATE VIEW <$view_name> AS <SELECT ...>;
```

# 2 视图的删除

```sql
-- 删除视图
DROP VIEW <$view_name>;
```

# 3 对视图中数据的修改

对于视图的 SELECT、INSERT、UPDATE 和 DELETE 操作和对表的操作是一样的，操作的结果会打到相应的表上。不过对视图操作的前提是 MySQL 能够确定到相应的数据，比如说使用了 `DISTINCT`s、`Sum()` 等之类的函数便不能对视图做 INSERT、UPDATE 和 DELETE 操作。

# 参考资料

1. [SQL VIEW](https://www.w3school.com.cn/sql/sql_view.asp)