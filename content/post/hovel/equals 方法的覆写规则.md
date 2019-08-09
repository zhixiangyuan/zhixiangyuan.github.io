---
title: "Equals 方法的覆写规则"
date: 2019-08-09T09:24:19+08:00
keywords: []
description: ""
tags: [

]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
draft: false
---

equals 方法有一些固定的规则，一些类（例如集合类）会按照规则使用 equals 方法，如果重写的时候不按照这些规则进行重写，那么在使用的时候就可能出现莫名其妙的 bug。对于 equals 方法，如果不需要重写就别重写，下面是四种不需要重写的情况：

1. 每一个类的实例都是与众不同的，那么就直接使用 Object 中的 equals 方法就好了。
2. 对于功能性的相等判断不要放到 equals 中
   - 比如对于 java.util.regex.Pattern 这个类，如果使用 equals 时对于两个不通的正则表达式，还去判断是否能起一样的效果的话，并不一定是一件好事。
3. 如果父类已经重写了，那么子类可以直接使用，不需要重写
4. 如果你确定这个类的 equals 方法永远不会被调用，也无需重写
   - 不过最好还是一下，在其中抛出异常，防止错误调用

# 1 重写 equals 需要满足的规则

1. 自反性：当 x != null 时，x.equals(x) must return true
2. 对称性：当 x != null & y != null 时，x.equals(y) return true，则 y.equals(x) 也必须 return true
3. 传递性：当 x, y, z != null 时，x.equals(y) return true and  y.equals(z) return true, then x.equals(z) must return true.
4. 一致性：对于一个没有被修改的类，多次 equals 方法调用的返回结果应该一致
5. x.equals(null) must return false

## 1.1 违反自反性的后果

```java
    public static void main(String[] args) {
        // 假设 BreakLow 违反了自反性，breakLow.equsls(breakLow) return false
        // 1 创建集合类并将其放进去
        List<BreakLow> breakLowList = new ArrayList<>();
        BreakLow breakLow = new BreakLow();
        breakLowList.add(breakLow);
        // 2 判断其是否在集合类当中
        // 这里就会 return false，这就很可怕，集合类直接就用不了了
        breakLowList.contains(breakLow);
    }
```