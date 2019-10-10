---
title: "Equals 方法的重写规则"
date: 2019-08-09T09:24:19+08:00
keywords: []
description: ""
tags: [
    "Java 基础"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

equals 方法有一些固定的规则，一些类（例如集合类）会按照规则使用 equals 方法，如果重写的时候不按照这些规则进行重写，那么在使用的时候就可能出现莫名其妙的 bug。对于 equals 方法，如果不需要重写就别重写，下面是四种不需要重写的情况：

1. 每一个类的实例都是与众不同的，那么就直接使用 Object 中的 equals 方法就好了。
2. 对于功能性的相等判断不要放到 equals 中
   - 比如对于 `java.util.regex.Pattern` 这个类，如果使用 equals 时对于两个不通的正则表达式，还去判断是否能起一样的效果的话，并不一定是一件好事。
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

## 1.2 违反对称性的后果

```java
// 这里以实现一个大小写不敏感的 String 为例
public class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    // 这个 equals 方法便违反了对称性原则
    @Override
    public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
        if (o instanceof String)
            return s.equalsIgnoreCase((String) o);
        return false;
    }

    public static void main(String[] args) {
        CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
        String s = "polish";
        List<CaseInsensitiveString> list = new ArrayList<>();
        list.add(cis);
        // 这里就出现了问题，虽然这是一个大小写不敏感的 String，但是将 string 
        // 直接传进去还是会返回 false
        System.out.println(list.contains(s));
    }
}
```

## 1.3 违反传递性的后果

违反传递性的情况会出现在子类继承一个可以实例化的父类，并在子类中增加字段的时候会出现这种情况，情况比较复杂，详细案例见 EffectiveJava

java 类库中的 `java.sql.TimeStamp` 与 `java.util.Date` 就不满足这个条件，所以不要将他们在集合类中混用，否则可能出现未知的 bug。

# 2 怎样写一个正确的 equals 

要么就别重写 equals，要么就让代码生成代码，毕竟人可能犯错，但是代码不会犯错。在 IDE 中使用自动生成 equsls 的功能，然后选择合适的参数去生成 equals。

# 参考资料

1. EffectiveJava