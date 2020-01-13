---
title: "查看 mysql 表相关信息"
date: 2020-01-13T15:30:42+08:00
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

```
-- \G 表示的意思是数据以列的方式显示
-- 需要注意的是在 navicat 的命令行里面这样用会报错

-- 通过以下语句查看表的相关信息

-- 查看所有表的信息
show table status \G

-- 查看某张表的信息（支持通配符）
show table status like '<$表名>' \G
```

每一项信息表示的含义

```
Name:
    表名
Engine:
    表的存储引擎类型。在旧版本中，该列的名字叫 Type，而不是 Engine。
Version：
    表的 .frm 文件的版本号
Row_format:
    行的格式。对于 MyISAM 表，可选值为 Dynamic、Fixed 或者 Compressed。Dynamic 的行
    长度是可变的，一般包含可变长度的字段，如 VARCHAR 或 BLOB。Fixed 的行长度则是固定
    的，只包含固定长度的列，如 CHAR 和 INTERGER。Compressed 的行则只在压缩表中存在。
Rows:
    表中的行数。对于 MyISAM 和其他一些存储引擎， 该值是精确的，但对于 InnoDB，该值是估计值。
Avg_row_length:
    平均每行包含的字节数。
Data_length:
    表数据的大小（以字节为单位）
Max_data_length:
    表数据的最大容量，该值和存储引擎有关。
Index_length:
    索引的大小（以字节为单位）
Data_free:
    对于 MyISAM 表，表示已分配但目前没有使用的空间。这部分空间包括了之前删除的行，以及
    后续可以被 INSERT 利用到的空间。
Auto_increment:
    下一个 AUTO_INCREMENT 的值。
Create_time:
    表的创建时间
Update_time:
    表数据的最后修改时间
Check_time:
    使用 CHECK TABLE 命令或者 myisamchk 工具最后一次检查表的时间
Collation:
    表的默认字符集和字符列排序规则。
Checksum:
    如果启用，保存的是整个表的实时校验和。
Create_options:
    创建表时指定的其他选项。
Comment:
    该列包含了一些其他的额外信息。对于 MyISAM 表，保存的是表在创建时带的注释。对于
    InnoDB 表，则保存的是 InnoDB 表空间的剩余空间信息。如果是一个视图，则该列包含
    “VIEW” 的文本字样。
```