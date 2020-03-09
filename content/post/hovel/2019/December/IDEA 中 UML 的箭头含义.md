---
title: "IDEA 中 UML 的箭头含义"
date: 2019-12-10T08:28:37+08:00
keywords: []
description: ""
tags: [
    "IntelliJ IDEA"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

本文中所使用的关系名词并不是 UML 中所定义的，因为我觉得他那种方式不够具体，对于 Javer 来说并不好理解，所以这里我直接用易于 javer 理解的名词来解释类之间的关系，以帮助开发看到 UML 就能大概知道内部的实现大概是什么样子。

# 1 实现关系

```java
public interface B {
}

public class A implements B {
}
```

这种关系在 IDEA 的 UML 中使用的是绿色实心虚线箭头

![A 实现了 B](/hub/2019/December/9.png)

# 2 继承关系

```java
public class B {
}

public class A extends B{
}
```

这种关系使用的是实线实心蓝色箭头

![class A extends B](/hub/2019/December/10.png)

```java
public interface B {
}

public interface A extends B{
}
```

接口之间的继承是实线实心绿色箭头

![interface A extends B](/hub/2019/December/11.png)

# 3 创建关系

```java
public class A {
    public A() {
         new B();
    }
}
```

这种虚线非实心箭头，上面还标了 create 的标志就是表示 A 类中某个位置创建了 B，这种同样可以理解为 A 调用了 B 的构造函数。

![A create B](/hub/2019/December/12.png)

# 4 调用关系

```java
public class A {
    public A(B b) {
         B bb = b;
    }
}
```

这种虚线实心箭头，上面没有标任何东西就是表示在 A 类中调用了 B，由于可以看到他们之间没有创建关系，而调用必须要用引用来调用，所以出现这种只有调用没有创建的时候，B 必然是以参数形式传入 A 的。

![](/hub/2019/December/13.png)

```java
public class B {
    public static void staticMethod() {
    }
}

public class A {
    public static void main(String[] args) {
        B.staticMethod();
    }
}
```

如果 A B 间存在方法调用同样是这种调用关系

![](/hub/2019/December/18.png)

```java
public class A {
    public A() {
         B b = new B();
    }
}
```

这种情况便是即有 new，又有调用的情况，不过这里只有一个引用指向 new 出来的 B，没有实际的调用，不过 IDEA 还是会归类于有调用关系。

![](/hub/2019/December/14.png)

```java
public class B {
    public void method() { }
}

public class A {
    public void method(B b) {
        b.method();
    }
}
```

这种同样是 A 调用了 B 的方法

![](/hub/2019/December/13.png)

# 5 持有关系

```java
public class A {
    private B b;
}
```

当 A 当中有 B 字段的时候，A 和 B 便是持有关系了，其实这种关系在设计模式里面叫包装模式，菱形箭头指向的是 master，非实心箭头指向的是 slave。下面的箭头两头都有 1，表示两边是一对一的关系。

![](/hub/2019/December/15.png)

```java
public class A {
    private List<B> b;
}
```

这是一种一对多的关系，图中 master 标着 1，slave 标着 *

![](/hub/2019/December/16.png)

# 6 注解相关

```java
public @interface B {
}

@B
public class A {

}
```

注解关系是用黄色虚线表示的

![](/hub/2019/December/19.png)

```java
public @interface B {
}

@B
public @interface A {

}
```

注解 B 标 A 同样用黄色虚线表示

![](/hub/2019/December/20.png)

# 7 内部类相关

```java
public class A {
    public class B {
    }
}
```

如果是内部类的形式，IDEA 的表示形式是下面这种形式

![](/hub/2019/December/17.png)