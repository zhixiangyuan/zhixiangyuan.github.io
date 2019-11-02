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
---

对于现在大多数的互联网应用，数据库的瓶颈在于数据库的读写，所以需要数据库尽快完成读写操作，业务操作放到服务器上做。

# 1 创建存储过程

```sql
DELIMITER $$
CREATE [DEFINER = {$user}] PROCEDURE <$procedure_name>
([[IN | OUT | INOUT] {$param_name} {$type}[,...]])
BEGIN
    
END$$
DELIMITER ;

-- 以下为加上注释的版本
-- DELIMITER 将语句的结束符号改成 $$ 防止和 ; 冲突
DELIMITER $$
-- [DEFINER = {$user}] 表示定义这个存储过程的用户，不写则默认成当前用户
-- <$function_name> 存储过程的名称
CREATE [DEFINER = {$user}] PROCEDURE <$function_name>
-- IN 输入参数：表示调用者向过程传入值（传入值可以是字面量或变量）
-- OUT 输出参数：表示过程向调用者传出值 (可以返回多个值)（传出值只能是变量）
-- INOUT 输入输出参数：既表示调用者向过程传入值，又表示过程向调用者传出值（值只能是变量）
([[IN | OUT | INOUT] {$param_name} {$tyspe}[,...]])
BEGIN
    
END$$
-- DELIMITER ; 将语句的结束符号改成 ;
DELIMITER ;
```

# 2 执行存储过程

```sql
CALL <$procedure_name>();
```

# 3 删除存储过程

```sql
DROP PROCEDURE <$procedure_name>;
```

