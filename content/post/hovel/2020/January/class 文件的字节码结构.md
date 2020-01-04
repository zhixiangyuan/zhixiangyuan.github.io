---
title: "class 文件的字节码结构"
date: 2020-01-04T11:41:22+08:00
keywords: []
description: ""
tags: [
    "java"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

# 1 结构预览

## 1.1 数据结构对照表

数据结构含义对照表，该数据结构将在下文中使用

| 数据结构       | 含义               |
| -------------- | ------------------ |
| u1             | 1 个字节的无符号数 |
| u2             | 2 个字节的无符号数 |
| u4             | 4 个字节的无符号数 |
| cp_info        | 常量信息           |
| field_info     | 字段信息           |
| method_info    | 方法信息           |
| attribute_info | 属性信息           |

class 文件字节码结构表，字节码结构顺序按照从上到下依次排列

| 字节码结构     | 含义                                 | 备注                        |
| -------------- | ------------------------------------ | --------------------------- |
| u4             | magic                                | 这个字段定死了是 0xCAFEBABE |
| u2             | minor_version                        | 副版本号                    |
| u2             | major_version                        | 主版本号                    |
| u2             | constant_pool_count                  | 常量池计数器                |
| cp_info        | constant_pool[constant_pool_count-1] | 常量池                      |
| u2             | access_flags                         | 访问标志                    |
| u2             | this_class                           | 类索引                      |
| u2             | super_class                          | 父类索引                    |
| u2             | interfaces_count                     | 接口计数器                  |
| u2             | interfaces[interfaces_count]         | 接口表                      |
| u2             | fields_count                         | 字段计数器                  |
| field_info     | fiels[fields_count]                  | 字段表                      |
| u2             | methods_count                        | 方法计数器                  |
| method_info    | methods[methods_count]               | 方法表                      |
| u2             | attributes_count                     | 属性计数器                  |
| attribute_info | attributes[attributes_count]         | 属性表                      |

## 1.2 magic

magic 称为魔数，唯一的作用是确定这个文件是否为一个能被虚拟机锁接收的 class 文件。魔数的值固定为 0xCAFEBABE，不会改变。

## 1.3 minor_version 和 major_version

minor_version 为副版本号，major_version 为主版本号，若 minor_version 为 `x`，major_version 为 `y`，则表示的版本号为 `y.x`。

## 1.4 constant_pool_count 和 constant_pool

constant_pool_count 的值等于常量池表中的成员数加 1.常量池表中的索引值只有在大于 0 且小于 constant_pool_count 时才会认为是有效的（即 0 < constant_pool.length < constant_pool_count），对于 long 和 double 类型有例外情况（todo 什么例外情况）

// todo 内容太多，敬请期待

# 2 参考文章

1. 《Java 虚拟机规范（Java SE 8 版）》