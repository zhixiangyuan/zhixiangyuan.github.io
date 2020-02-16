---
title: "修复 Google Guava 框架的 CaseFormat 类中的 Bug"
date: 2020-02-16T18:50:21+08:00
keywords: []
description: ""
tags: [
    "开源贡献"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

CaseFormat 可以很方便地实现字符串驼峰转下划线的功能，内部的实现原理是寻找需要做转换的字母处，然后将其字母前方的单词做截取，再根据转换后的字符串需求做拼接。

而我发现发现的 Bug 便于拼接有关，它内部使用 StringBuilder 来实现拼接字符串，在 `new StringBuilder` 时会申请一块 `char[]` 的连续空间，这个地方它做了一个小优化，由于驼峰转下划线，那么必然字符串会变长，所以它在申请的时候额外申请 `4 * 下划线长度` 空间用于存放下划线，但是它在编写代码的时候使用错了字段，本来应该用转换之后的分隔符，结果用成了转换之前的分隔符，我对这里做了一下修改然后提交到官方仓库，最终被官方接受并合并。

```java
    // 原来的代码片段
    out = new StringBuilder(s.length() + 4 * wordSeparator.length());
    // 修改后的代码片段
    out = new StringBuilder(s.length() + 4 * format.wordSeparator.length());
```

[此处是合并记录的链接](https://github.com/google/guava/commit/b4a573fae2a1b4c78652dba3a59da72ff6b80a13#diff-a14d49e99a35efab903b85d6ea070834)