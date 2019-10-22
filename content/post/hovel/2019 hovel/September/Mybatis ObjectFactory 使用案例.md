---
title: "MyBatis ObjectFactory 使用案例"
date: 2019-09-10T07:02:15+08:00
keywords: []
description: ""
tags: [
    "MyBatis"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

MyBatis 每次创建结果对象的新实例时，它都会使用一个对象工厂来完成。默认的对象工厂仅仅是实例化目标类，要么通过默认构造方法，要么在构造方法上参数映射存在的时候通过参数构造方法来实例化。所以，如果我们希望在查询结果对象转换为 Java model 的时候做一些自己的操作，就可以通过自定义对象工厂的方式来完成。

# 1 编写自定义 ObjectFactory 对象

MyBatis 中有默认的对象工厂，我们只需要继承并重写需要重写的方法即可。下面的代码目的是为了在创建 Dept 对象的时候为其中的 country 字段赋值 China。

```java
public class MyObjectFactory extends DefaultObjectFactory {

    @Override
    public Object create(Class type) {
        if (Dept.class == type) {
            //依靠父类提供 create 方法创建 Dept 实例对象
            Dept dept = super.<Dept>create(type);
            //设置自定义规则
            dept.setCountry("China");
            return dept;
        }
        return super.create(type);
    }
}
```
# 2 在配置文件中注册该对象工厂

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    ...
    <objectFactory type="me.yuanzx.demo.util.MyObjectFactory"/>
    ...
</configuration>
```

# 3 测试看是否生效

```java
public class TestMain {
    private SqlSession session;

    @Before
    public void start() {
        try {
            InputStream inputStream = Resources.getResourceAsStream("myBatis-config.xml");
            SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(inputStream);
            session = factory.openSession();
        } catch (Exception exception) {
            exception.printStackTrace();
        }
    }

    @Test
    public void test() {
        DeptMapper dao = session.getMapper(DeptMapper.class);
        List<Dept> deptList = dao.deptFind();
        for (Dept dept : deptList) {
            System.out.println(dept);
        }
    }

    @After
    public void end() {
        if (session != null) {
            session.close();
        }
    }
}
```

# 4 稍微看看 DefaultObjectFactory 源码

```java
// 这个类还是挺简单的，没有 N 个父类，仅仅是实现了两个接口
// 其中 ObjectFactory 接口定义了对象工厂需要干的活
public class DefaultObjectFactory implements ObjectFactory, Serializable {

  private static final long serialVersionUID = -8855120656740914948L;

  @Override
  public <T> T create(Class<T> type) {
    return create(type, null, null);
  }

  @SuppressWarnings("unchecked")
  @Override
  public <T> T create(Class<T> type, List<Class<?>> constructorArgTypes, List<Object> constructorArgs) {
    Class<?> classToCreate = resolveInterface(type);
    // we know types are assignable
    return (T) instantiateClass(classToCreate, constructorArgTypes, constructorArgs);
  }

  @Override
  public void setProperties(Properties properties) {
    // no props for default
  }

  // 实例化实例在这个里面完成
  private  <T> T instantiateClass(Class<T> type, List<Class<?>> constructorArgTypes, List<Object> constructorArgs) {
    try {
      Constructor<T> constructor;
      if (constructorArgTypes == null || constructorArgs == null) {
        // 如果构造参数类型列表或者是构造参数为 null
        // 则直接用默认构造函数创建对象
        constructor = type.getDeclaredConstructor();
        if (!constructor.isAccessible()) {
          // 如果默认构造函数不可访问则将其修改为可访问
          constructor.setAccessible(true);
        }
        return constructor.newInstance();
      }
      // 走到这一步，则说明 constructorArgTypes != null && constructorArgs != null
      // 那么则通过参数列表的类型去寻找构造函数
      // 然后通过该构造函数创建对象
      constructor = type.getDeclaredConstructor(constructorArgTypes.toArray(new Class[constructorArgTypes.size()]));
      if (!constructor.isAccessible()) {
        constructor.setAccessible(true);
      }
      return constructor.newInstance(constructorArgs.toArray(new Object[constructorArgs.size()]));
    } catch (Exception e) {
      StringBuilder argTypes = new StringBuilder();
      if (constructorArgTypes != null && !constructorArgTypes.isEmpty()) {
        for (Class<?> argType : constructorArgTypes) {
          argTypes.append(argType.getSimpleName());
          argTypes.append(",");
        }
        argTypes.deleteCharAt(argTypes.length() - 1); // remove trailing ,
      }
      StringBuilder argValues = new StringBuilder();
      if (constructorArgs != null && !constructorArgs.isEmpty()) {
        for (Object argValue : constructorArgs) {
          argValues.append(String.valueOf(argValue));
          argValues.append(",");
        }
        argValues.deleteCharAt(argValues.length() - 1); // remove trailing ,
      }
      throw new ReflectionException("Error instantiating " + type + " with invalid types (" + argTypes + ") or values (" + argValues + "). Cause: " + e, e);
    }
  }
  
  // 这里面主要在对一些接口去赋值实现类
  protected Class<?> resolveInterface(Class<?> type) {
    Class<?> classToCreate;
    if (type == List.class || type == Collection.class || type == Iterable.class) {
      // 如果 type 是 List、Collection、Iterable 则创建 ArrayList 类型
      classToCreate = ArrayList.class;
    } else if (type == Map.class) {
      // 如果 type 是 Map 则创建 HashMap
      classToCreate = HashMap.class;
    } else if (type == SortedSet.class) { // issue #510 Collections Support
      // 如果 type 是 SortedSet 则创建 TreeSet
      classToCreate = TreeSet.class;
    } else if (type == Set.class) {
      // 如果 type 是 Set 则创建 HashSet
      classToCreate = HashSet.class;
    } else {
      // 如果不是上述类型，则直接创建 type 类型
      classToCreate = type;
    }
    return classToCreate;
  }

  @Override
  public <T> boolean isCollection(Class<T> type) {
    return Collection.class.isAssignableFrom(type);
  }

}
```