---
title: "double 转 BigDecimal 造成的精度丢失"
date: 2019-07-10T16:07:14+08:00
keywords: []
description: ""
tags: [

]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

```java
    public static void main(String[] args) {
        double d1 = 36.8d;
        double d2 = 36.5d;
        BigDecimal subtract = new BigDecimal(d1).subtract(new BigDecimal(d2));
        System.out.println(subtract);
    }
```

对于上述类型的计算会得到结果 0.2999999999999971578290569595992565155029296875，这明显出现了精度丢失，这是由于使用 BigDecimal 的方式错误导致的。正确的使用方式是下面这样。

```java
    public static void main(String[] args) {
        double d1 = 36.8d;
        double d2 = 36.5d;
        BigDecimal subtract = new BigDecimal(Double.toString(d1))
                .subtract(new BigDecimal(Double.toString(d2)));
        System.out.println(subtract);
    }
```

这样便能计算出正确结果 0.3，同样可以用 BigDecimal.valueOf(double)，它里面也是将 double 转成 String 来计算的。