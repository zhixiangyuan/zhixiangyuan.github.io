---
title: "Unsafe 解析"
date: 2019-07-23T20:29:17+08:00
keywords: []
description: ""
tags: [
    "Java 并发编程"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

>本文基于 jdk1.8

Unsafe 位于 sun.misc 包下，提供能直接访问系统内存资源的方法，使得用户能够自主管理内存，使用得当能够提升程序的运行效率。

# 1 如何获取 Unsafe 实例

下面代码为 Unsafe 源码，可以看到 Unsafe 内部是单例模式实现，生成的单例放在 theUnsafe，同时提供了一个 `getUnsafe()` 的方法给外部调用。

```java
public final class Unsafe {
    ...
    private static final Unsafe theUnsafe;

    private Unsafe() {
    }

    @CallerSensitive
    public static Unsafe getUnsafe() {
        Class var0 = Reflection.getCallerClass();
        if (!VM.isSystemDomainLoader(var0.getClassLoader())) {
            throw new SecurityException("Unsafe");
        } else {
            return theUnsafe;
        }
    }

    static {
        ...
        theUnsafe = new Unsafe();
        ...
    }
    ...
}
```

虽然其中提供了 `getUnsafe()` 的方法给外部调用，但是这句代码 `VM.isSystemDomainLoader(var0.getClassLoader())` 同样限制了，如果不是系统域加载器加载的类，是无法调用这个方法的，一旦调用，就会报一个不安全的异常。所以想要获取 Unsafe，就通过下面这个反射的方式将其拿出来。其中的 `field.get(null);` 这句代码，由于 get 的是静态变量，所以可以直接传 null 进去。

```java
    public static Unsafe reflectGetUnsafe() {
        try {
            Field field = Unsafe.class.getDeclaredField("theUnsafe");
            field.setAccessible(true);
            return (Unsafe) field.get(null);
        } catch (IllegalAccessException | NoSuchFieldException e) {
            log.error(e.getMessage(), e);
            return null;
        }
    }
```

# 2 Unsafe 的功能

![Unsafe 的功能](/media/hovel/26.png)

如上图所示，Unsafe 提供的 API 大致可分为内存操作、CAS、Class 相关、对象操作、线程调度、系统信息获取、内存屏障、数组操作，下面进行具体介绍。

## 2.1 内存操作

```java
    /**
     * 申请分配内存
     * 
     * @param capacity 申请的内存大小
     * @return 内存地址
     */
    public native long allocateMemory(long capacity);

    /**
     * 重新分配内存，这个方法将会释放掉给定内存地址所使用的内存，然后重新
     * 申请给定大小的内存。由于其会释放原来的内存，所以如果有值需要用，调
     * 用这个方法之前，先把其中的值拿出来，然后再调用这个方法
     *
     * @param address 内存地址
     * @param bytes   申请的内存大小
     */
    public native long reallocateMemory(long address, long bytes);

    /**
     * 释放内存
     * 
     * @param address 内存地址
     */
    public native void freeMemory(long address);

    /**
     * 内存块的地址由对象引用 o 和偏移地址共同决定，如果对象引用为 null，那么
     * offset 就是绝对地址。capacity 的大小和申请的内存地址大小一样即可。value 
     * 会将申请的内存全部置为 value。比如申请两个字节内存，value 填 0b00000001，
     * 那么申请的两个直接会被初始化为 0b00000001_00000001
     *
     * @param o        对象引用
     * @param offset   偏移地址
     * @param capacity 内存块大小，与申请的内存块大小相同即可
     * @param value    初始值，申请的内存块每个字节都会初始化 value
     */
    public native void setMemory(Object o, long offset, long capacity, byte value);
```

## 2.2 CAS 

```java
    /**
    * 首先通过对象引用和 offset 找到需要更新的值的地址，在更新的时候，首先检查
    * 内存中的值是否等于 expected 如果相等，则修改为 update，修改成功，返回 true；
    * 如果不等，则修改失败，返回 false。
    * 
    * @param o         包含要修改 field 的对象
    * @param offset    对象中某 field 的偏移量
    * @param expected  期望值
    * @param update    更新值
    * @return          true 更新成功 | false 更新失败
    */
    public final native boolean compareAndSwapObject(Object o, long offset,  Object expected, Object update);

    /**
    * {@link Unsafe#compareAndSwapObject(Object, long, Object, Object)}
    * 
    * @param o         包含要修改 field 的对象
    * @param offset    对象中某 field 的偏移量
    * @param expected  期望值
    * @param update    更新值
    * @return          true 更新成功 | false 更新失败
    */
    public final native boolean compareAndSwapInt(Object o, long offset, int expected,int update);
    
    /**
    * {@link Unsafe#compareAndSwapObject(Object, long, Object, Object)}
    * 
    * @param o         包含要修改 field 的对象
    * @param offset    对象中某 field 的偏移量
    * @param expected  期望值
    * @param update    更新值
    * @return          true 更新成功 | false 更新失败
    */
    public final native boolean compareAndSwapLong(Object o, long offset, long expected, long update);
```

## 2.3 Class 相关

```java
    /**
     * 获取给定静态字段的内存地址偏移量，这个值对于给定的字段是唯一且固定不变的
     *
     * @param field 静态字段
     */
    public native long staticFieldOffset(Field field);

    /** 
     * 获取一个静态类中给定字段的类的引用
     *
     * @param field 静态字段
     */
    public native Object staticFieldBase(Field field);

    /**
     * 判断是否需要初始化一个类，通常在获取一个类的静态属性的时候
     * （因为一个类如果没初始化，它的静态属性也不会初始化）使用。
     *  当且仅当 ensureClassInitialized 方法不生效时返回 false。
     * 
     * @param clazz
     */
    public native boolean shouldBeInitialized(Class<?> clazz);

    /** 
     * 检测给定的类是否已经初始化。通常在获取一个类的静态属性的时候
     * （因为一个类如果没初始化，它的静态属性也不会初始化）使用。
     * 
     * @param clazz
     */
    public native void ensureClassInitialized(Class<?> clazz);

    /**
     * 定义一个类，此方法会跳过 JVM 的所有安全检查，默认情况下，ClassLoader（类加载器）
     * 和 ProtectionDomain（保护域）实例来源于调用者
     *
     * @param name
     * @param b
     * @param off
     * @param len
     * @param loader
     * @param protectionDomain
     */
    public native Class<?> defineClass(String name, byte[] b, int off, int len, ClassLoader loader, ProtectionDomain protectionDomain);

    /**
     * 定义一个匿名类
     * 
     * @param hostClass
     * @param data
     * @param cpPatches
     */
    public native Class<?> defineAnonymousClass(Class<?> hostClass, byte[] data, Object[] cpPatches);
```

## 2.4 对象操作

```java
    /**
     * 返回对象成员属性在内存地址相对于此对象的内存地址的偏移量
     *
     * @param field
     */
    public native long objectFieldOffset(Field field);

    /**
     * 获得给定对象的指定地址偏移量的值
     *
     * @param o
     * @param offset
     */
    public native Object getObject(Object o, long offset);

    /**
     * 给定对象的指定地址偏移量设值
     *
     * @param o
     * @param offset
     * @param x
     */
    public native void putObject(Object o, long offset, Object x);

    /**
     * 从对象的指定偏移量处获取变量的引用，使用 volatile 的加载语义
     *
     * @param o
     * @param offset
     */
    public native Object getObjectVolatile(Object o, long offset);

    /**
     * 存储变量的引用到对象的指定的偏移量处，使用 volatile 的存储语义
     *
     * @param o
     * @param offset
     * @param x
     */
    public native void putObjectVolatile(Object o, long offset, Object x);

    /**
     * 有序、延迟版本的 putObjectVolatile 方法，不保证值的改变被其他线程立即看到。
     * 只有在field被volatile修饰符修饰时有效
     *
     * @param o
     * @param offset
     * @param x
     */
    public native void putOrderedObject(Object o, long offset, Object x);

    /**
     * 绕过构造方法、初始化代码来创建对象
     *
     * @param clazz
     */
    public native Object allocateInstance(Class<?> clazz) throws InstantiationException;

    /**
     * 向指定内存中写入一个 byte 
     *
     * @param address 内存地址
     * @param value   byte 
     */
    public native void putByte(long address, byte value);

    /**
     * 从指定内存中取出一个 byte 数
     *
     * @param address 内存地址
     */
    public native byte getByte(long address);

    /**
     * 向指定内存中写入一个 char 
     *
     * @param address 内存地址
     * @param value   char
     */
    public native void putChar(long address, char value);

    /**
     * 从指定内存中取出一个 char
     *
     * @param address 内存地址
     */
    public native char getChar(long address);

    /**
     * 向指定内存中写入一个 short 数
     *
     * @param address 内存地址
     * @param value   short
     */
    public native void putShort(long address, short value);

    /**
     * 从指定内存中取出一个 short 数
     *
     * @param address 内存地址
     */
    public native short getShort(long address);

    /**
     * 向指定内存中写入一个 int 数
     *
     * @param address 内存地址
     * @param value   int
     */
    public native int getInt(long address);

    /**
     * 从指定内存中取出一个 int 数
     *
     * @param address 内存地址
     */
    public native void putInt(long address, int value);

    /**
     * 向指定内存中写入一个 long 数
     *
     * @param address 内存地址
     * @param value   long
     */
    public native int getLong(long address);

    /**
     * 从指定内存中取出一个 long 数
     *
     * @param address 内存地址
     */
    public native void putLong(long address, long value);

    /**
     * 向指定内存中写入一个 float 数
     *
     * @param address 内存地址
     * @param value   float
     */
    public native int getFloat(long address);

    /**
     * 从指定内存中取出一个 float 数
     *
     * @param address 内存地址
     */
    public native void putFloat(long address, float value);

    /**
     * 向指定内存中写入一个 double 数
     *
     * @param address 内存地址
     * @param value   double
     */
    public native int getDouble(long address);

    /**
     * 从指定内存中取出一个 double 数
     *
     * @param address 内存地址
     */
    public native void putDouble(long address, double value);
```

## 2.5 线程调度

```java
    /**
     * 阻塞线程，当 isAbsolute 为 false 时，time 为等待时间，时间单位
     * 是纳秒，1 秒等 1000_000_000 纳秒；当 isAbsolute 为 true 时，time
     * 为 unix 时间戳，表示一直等到 unix 时间戳的时间才释放
     *
     * @param isAbsolute 与 time 连用
     * @param time 
     */
    public native void park(boolean isAbsolute, long time);

    /**
     * 取消阻塞线程
     * 
     * @param thread 取消阻塞该线程
     */
    public native void unpark(Object thread);
```

## 2.6 系统信息

```java
    /**
     * 返回系统指针的大小。返回值为4（32位系统）或 8（64位系统）。
     */
    public native int addressSize();

    /**
     * 获取本地内存的页数，此值为 2 的幂次方
     */
    public native int pageSize();
```

## 2.7 内存屏障

```java
    /**
     * 内存屏障，禁止 load 操作重排序。屏障前的 load 操作不能被重排序到屏障后，
     * 屏障后的 load 操作不能被重排序到屏障前
     */
    public native void loadFence();
    /**
     * 内存屏障，禁止 store 操作重排序。屏障前的 store 操作不能被重排序到屏障后，
     * 屏障后的 store 操作不能被重排序到屏障前
     */
    public native void storeFence();
    /**
     * 内存屏障，禁止 load、store 操作重排序
     */
    public native void fullFence();
```

## 2.8 数组操作

```java
    /**
     * 返回数组中第一个元素的偏移地址
     * 
     * @param arrayClass 数组类，比如 int[].class
     */
    public native int arrayBaseOffset(Class<?> arrayClass);
    /**
     * 返回数组中一个元素占用的大小
     * 
     * @param arrayClass 数组类，比如 int[].class
     */
    public native int arrayIndexScale(Class<?> arrayClass);
```

# 参考资料

1. [Java 魔法类：Unsafe 应用解析](https://tech.meituan.com/2019/02/14/talk-about-java-magic-class-unsafe.html)