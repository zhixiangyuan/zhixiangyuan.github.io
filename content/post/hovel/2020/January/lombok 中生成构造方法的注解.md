---
title: "lombok 中生成构造方法的注解"
date: 2020-01-06T19:50:07+08:00
keywords: []
description: ""
tags: [
    "lombok"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

> 本文基于 org.projectlombok.lombok:1.18.10 

这样的注解总共有三个，分别是 @NoArgsConstructor、@RequiredArgsConstructor 和 @AllArgsConstructor。

# 1 @NoArgsConstructor

先看一下注解的源码

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.SOURCE)
public @interface NoArgsConstructor {
    /** 生成一个静态类去创建对象，具体效果看下面的例子 */
    String staticName() default "";

    /**
     * 指定添加到生成的构造函数上的注解，支持两种语法
     * @NoArgsConstructor(onConstructor=@__({@AnnotationsGoHere})) 
     * 或者是 @NoArgsConstructor(onConstructor_={@AnnotationsGoHere})
     * 这里之所以提供新的语法是因为如果不这样，那么这里函数的返回值就要写成 Class<?>
     * 可能 lombok 觉得这样不够优雅 \笑哭
     */
    AnyAnnotation[] onConstructor() default {};

    /** 设置构造函数的访问权限，默认为 PUBLIC */
    AccessLevel access() default lombok.AccessLevel.PUBLIC;

    /** 初始化 final 字段为 0/null/false 防止编译时错误 */
    boolean force() default false;

    @Deprecated
    @Retention(RetentionPolicy.SOURCE)
    @Target({})
    @interface AnyAnnotation {}
}
```

下面用例子展示实际效果

```java
// 1. 直接使用该注解的效果
@NoArgsConstructor
public class Message {
    private Long id;
}
// 以下是生成的代码
public class Message {
    private Long id;

    public Message() {}
}

// 2. 演示 staticName 参数的作用
@NoArgsConstructor(staticName = "createInstance")
public class Message {
    private Long id;
}
// 以下是生成的代码
public class Message {
    private Long id;

    private Message() {}

    public static Message createInstance() {
        return new Message();
    }
}

// 3. 演示 onConstructor 的作用
// 先定义一个自定义注解
@Target(ElementType.TYPE_USE)
@Retention(RetentionPolicy.RUNTIME)
@interface Custom {}
// 然后使用
@NoArgsConstructor(onConstructor_ = {@Custom})
public class Message {
    private Long id;
}
// 以下是生成的代码
public class Message {
    private Long id;

    @Custom
    public Message() {}
}

// 4. 演示 access 的作用
@NoArgsConstructor(access = AccessLevel.PRIVATE)
public class Message {
    private Long id;
}
// 以下是生成的代码
public class Message {
    private Long id;

    private Message() {}
}

// 5. 演示 force 的作用
@NoArgsConstructor(force = true)
public class Message {
    private final long id1;
    private final Long id2;
    private final boolean flag;
}
// 以下是生成的代码
public class Message {
    private final long id1 = 0L;
    private final Long id2 = null;
    private final boolean flag = false;

    public Message() {}
}

```

# 2 @RequiredArgsConstructor

先看一下代码

```java
/** 对加了 @NonNull 和 final 的字段进行初始化 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.SOURCE)
public @interface RequiredArgsConstructor {
    /** 创建一个静态工厂来创建对象，构造参数列表和静态工厂参数列表相同*/
    String staticName() default "";

    /** 同 NoArgsConstructor 不再赘述*/
    AnyAnnotation[] onConstructor() default {};

    /** 修改构造函数的权限修饰符*/
    AccessLevel access() default lombok.AccessLevel.PUBLIC;

    @Deprecated
    @Retention(RetentionPolicy.SOURCE)
    @Target({})
    @interface AnyAnnotation {}
}
```

以下是实际的演示案例

```java
@RequiredArgsConstructor(
        staticName = "createInstance",
        access = AccessLevel.PRIVATE
)
public class Message {
    private final long id1;
    @NonNull
    private Long id2;
    private boolean flag;
}
// 以下是生成的代码
public class Message {
    private final long id1;
    @NonNull
    private Long id2;
    private boolean flag;
    
    // 注意这里没有 flag 参数
    private Message(long id1, @NonNull Long id2) {
        if (id2 == null) {
            throw new NullPointerException("id2 is marked non-null but is null");
        } else {
            this.id1 = id1;
            this.id2 = id2;
        }
    }

    private static Message createInstance(long id1, @NonNull Long id2) {
        return new Message(id1, id2);
    }
}
```

# 3 @AllArgsConstructor

先看一下源码

```java
/** 生成包含所有参数的构造函数 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.SOURCE)
public @interface AllArgsConstructor {
    String staticName() default "";

    AnyAnnotation[] onConstructor() default {};

    AccessLevel access() default lombok.AccessLevel.PUBLIC;

    @Deprecated
    @Retention(RetentionPolicy.SOURCE)
    @Target({})
    @interface AnyAnnotation {}
}
```

以下是案例

```java
@AllArgsConstructor(
        staticName = "createInstance",
        access = AccessLevel.PRIVATE
)
public class Message {
    private final long id1;
    @NonNull
    private Long id2;
    private boolean flag;
}
// 以下是生成的代码
public class Message {
    private final long id1;
    @NonNull
    private Long id2;
    private boolean flag;

    // 可以看到这里包含所有参数
    private Message(long id1, @NonNull Long id2, boolean flag) {
        if (id2 == null) {
            throw new NullPointerException("id2 is marked non-null but is null");
        } else {
            this.id1 = id1;
            this.id2 = id2;
            this.flag = flag;
        }
    }

    private static Message createInstance(long id1, @NonNull Long id2, boolean flag) {
        return new Message(id1, id2, flag);
    }
}
```