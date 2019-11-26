---
title: "WeakReference 的使用场景"
date: 2019-11-26T11:33:07+08:00
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

WeakReference 所指的便是弱引用，它的作用便是能够让你获取到引用到对象，但是当弱引用引用的对象没有强引用引用时，那么便可以被虚拟机回收。这个类可以用在下面这种场景，这是我在安卓同学的代码里面见到的一种场景。

```java
class A {
    private B b;

    public A setB(B b) {
        this.b = b;
        return this;
    }
}

class B {
    private A a;

    public B setA(A a) {
        this.a = a;
        return this;
    }
}

public class Main {
    public static void main(String[] args) {
        A a = new A();
        B b = new B();
        a.setB(b);
        b.setA(a);
    }
}
```

可以看到，上面这种代码便构造出了一个循环引用的情况，在安卓里面，如果出现了这种循环引用，那么老版本的安卓虚拟机可能使用的是引用计数法判断对象的生死，那么便无法清理这种循环引用的对象，那么便会造成内存泄露。而如果将其改成使用 WeakReference，那么便不会出现循环引用，哪怕是使用引用计数法判断对象生死的虚拟机也可以将其清理掉。

```java
class A {
    private WeakReference<B> b;

    public A setB(B b) {
        this.b = new WeakReference<>(b);
        return this;
    }
}

class B {
    private A a;

    public B setA(A a) {
        this.a = a;
        return this;
    }
}

public class Main {
    public static void main(String[] args) {
        A a = new A();
        B b = new B();
        a.setB(b);
        b.setA(a);
    }
}
```

可以看到，同样的代码，我们只是在 A 中加入弱引用，那么现在 b 强引用了 a，但是 a 弱引用了 b，这样在虚拟机运行的时候便能够处理这种情况。