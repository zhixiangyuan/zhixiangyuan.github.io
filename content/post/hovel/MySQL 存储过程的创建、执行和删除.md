---
title: "MySQL 存储过程的创建、执行和删除"
date: 2019-11-01T18:39:30+08:00
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

# 1 创建存储过程

```sql
CREATE PROCEDURE productpricing()
BEGIN
    SELECT Avg(prod_price) AS price_average FROM products;
END;
```

# 2 执行存储过程

# 3 删除存储过程