---
title: "Mybatis Plugin 使用案例"
date: 2019-09-10T07:34:23+08:00
keywords: []
description: ""
tags: [
    "Mybatis"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

Mybatis 插件，实现功能的插件又可称之为拦截器，主要提供对 org.apache.ibatis.executor.Executor、org.apache.ibatis.executor.statement.StatementHandler、org.apache.ibatis.executor.parameter.ParameterHandler、org.apache.ibatis.executor.resultset.ResultSetHandler 中定义的方法做拦截扩展，首先写一个拦截起，看看他是怎么定位到拦截到哪个类的哪个方法的

# 1 实现一个拦截器

拦截器的实现通过在 Intercepts 注解中定义需要代理的接口（type），以及接口中的方法名（method）和参数列表（args），通过这几个参数就可以确定到需要代理哪个方法，我们需要实现的功能

```java
@Intercepts({
        // 这里是数组，可以拦截多个方法
        @Signature(
                // 以下三个也可以被代理:
                // 1. StatementHandler.class
                // 2. ParameterHandler.class
                // 3. ResultSetHandler.class
                type = Executor.class,
                // 方法名称
                method = "query",
                // 下面是 Executor 中的 query 方法的参数列表
                args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class}
        )
})
public class SimpleInterceptor implements Interceptor {

    /**
     * Invocation 对象是 Mybatis 中封装的类，这个类当中封装了代理对象、需要拦截的方法和
     * 该方法的参数列表
     */
    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        System.out.println("在拦截方法执行前做操作");
        //执行被拦截方法
        Object object = invocation.proceed();
        System.out.println("在拦截方法执行后做操作");
        return object;
    }

    /**
     * 这个方法的目的也就是对传入的对象进行包装，一般的写法也就是 
     * Plugin.wrap(target, this)，通过 Mybatis 的 Plugin 对其进行
     * 包装
     */
    @Override
    public Object plugin(Object target) {
        return Plugin.wrap(target, this);
    }

    @Override
    public void setProperties(Properties properties) {
        // TODO Auto-generated method stub

    }
}

public class Invocation {

  private final Object target;
  private final Method method;
  private final Object[] args;

  public Invocation(Object target, Method method, Object[] args) {
    this.target = target;
    this.method = method;
    this.args = args;
  }

  public Object getTarget() {
    return target;
  }

  public Method getMethod() {
    return method;
  }

  public Object[] getArgs() {
    return args;
  }

  public Object proceed() throws InvocationTargetException, IllegalAccessException {
    return method.invoke(target, args);
  }

}
```

# 2 在 Mybatis 中注册该拦截器

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    ... 
    <plugins>
        <plugin interceptor="me.yuanzx.demo.util.SimpleInterceptor"/>
    </plugins>
    ...
</configuration>
```

# 3 运行测试，查看效果

由于我们代理的是 query 这个方法在调用 select 的 sql 时会触发，所以这里我们只需要写一个 select 的 sql 去测试就能看到效果。

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
        // 这里只要是 select 的 sql 就能触发该拦截器
        List<Dept> deptList = dao.deptFind();
        System.out.println();
    }

    @After
    public void end() {
        if (session != null) {
            session.close();
        }
    }

}
```

# 4 搂一搂 Plugin 的源码

```java
// Plugin 实现了 InvocationHandler 可以看出来是一个代理类的实现方法
// 这里看不懂的同学需要先学习一下 Java 的动态代理，然后再往下看
public class Plugin implements InvocationHandler {

  private final Object target;
  private final Interceptor interceptor;
  private final Map<Class<?>, Set<Method>> signatureMap;

  private Plugin(Object target, Interceptor interceptor, Map<Class<?>, Set<Method>> signatureMap) {
    this.target = target;
    this.interceptor = interceptor;
    this.signatureMap = signatureMap;
  }
  // 我们在对 对象 进行包装时使用了这个方法，进来看一下
  // 这里的 target 是 Mybatis 传给我们的对象
  // interceptor 是我们自己实现的拦截器
  public static Object wrap(Object target, Interceptor interceptor) {
    // getSignatureMap 这个方法就是将我们拦截器上面的方法签名摘下来
    // 其中存放的 key 是 type，value 是 method + args 组成的方法
    Map<Class<?>, Set<Method>> signatureMap = getSignatureMap(interceptor);
    Class<?> type = target.getClass();
    // 这个方法会拿出 type 的所有接口，然后看这些接口是否在 signatureMap
    // 的 key 当中，如果有哪个接口在 key 中存在，则将该接口的 class 保存下来
    // 返回给我们，也就是 interfaces
    Class<?>[] interfaces = getAllInterfaces(type, signatureMap);
    if (interfaces.length > 0) {
      // 如果传入的 target 含有我们想要代理的类，则会对该类进行代理
      return Proxy.newProxyInstance(
          type.getClassLoader(),
          interfaces,
          new Plugin(target, interceptor, signatureMap));
    }
    return target;
  }

  @Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    try {
      Set<Method> methods = signatureMap.get(method.getDeclaringClass());
      if (methods != null && methods.contains(method)) {
        // 这里判断出来需要代理，则会使用我们的拦截器来调用方法
        return interceptor.intercept(new Invocation(target, method, args));
      }
      return method.invoke(target, args);
    } catch (Exception e) {
      throw ExceptionUtil.unwrapThrowable(e);
    }
  }

  // 这里的 interceptor 是我们实现的拦截器，这个方法就是将我们写在 interceptor 上的
  // 注解摘下来，然后根据我们在上面指定的那几个参数找到我们想要代理的方法，然后将其
  // 存成一个 Map，Map 的 key 是我们指定的 type，value 中的 set 中存放了我们想要代理
  // 的方法。
  private static Map<Class<?>, Set<Method>> getSignatureMap(Interceptor interceptor) {
    Intercepts interceptsAnnotation = interceptor.getClass().getAnnotation(Intercepts.class);
    // issue #251
    if (interceptsAnnotation == null) {
      throw new PluginException("No @Intercepts annotation was found in interceptor " + interceptor.getClass().getName());      
    }
    Signature[] sigs = interceptsAnnotation.value();
    Map<Class<?>, Set<Method>> signatureMap = new HashMap<Class<?>, Set<Method>>();
    for (Signature sig : sigs) {
      Set<Method> methods = signatureMap.get(sig.type());
      if (methods == null) {
        methods = new HashSet<Method>();
        signatureMap.put(sig.type(), methods);
      }
      try {
        Method method = sig.type().getMethod(sig.method(), sig.args());
        methods.add(method);
      } catch (NoSuchMethodException e) {
        throw new PluginException("Could not find method on " + sig.type() + " named " + sig.method() + ". Cause: " + e, e);
      }
    }
    return signatureMap;
  }

  // 这个方法会将 type 中所有的接口都拿出来，然后看看这个接口在不在 signatureMap 
  // 的 key 当中，如果存在，则将该接口保存起来然后返回
  private static Class<?>[] getAllInterfaces(Class<?> type, Map<Class<?>, Set<Method>> signatureMap) {
    Set<Class<?>> interfaces = new HashSet<Class<?>>();
    while (type != null) {
      for (Class<?> c : type.getInterfaces()) {
        if (signatureMap.containsKey(c)) {
          interfaces.add(c);
        }
      }
      type = type.getSuperclass();
    }
    return interfaces.toArray(new Class<?>[interfaces.size()]);
  }

}
```