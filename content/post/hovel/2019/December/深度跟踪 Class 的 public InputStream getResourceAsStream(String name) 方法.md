---
title: "深度跟踪 Class 的 public InputStream getResourceAsStream(String name) 方法"
date: 2019-12-07T06:31:41+08:00
keywords: []
description: ""
tags: [
    "java"
]
categories: [
    "杂货铺"
]
autoCollapseToc: true
author: "yuanzx"
---

>本日志基于 openjdk8

# 1 public final class Class<T>

我们从这个方法开始

```java
    public InputStream getResourceAsStream(String name) {
        // 这个方法见 1.1 解析
        // 这里假如传入的 name 为 /Web.xml，那么解析出来为 Web.xml，仅仅去掉了 / 就出来了
        // 如果这里传入的是 Web.xml，那么就会去寻找是哪个类调用的这个方法，比如说 String 调用的
        // String.class.getResourceAsStream("Web.xml")，那么他就会返回 java/lang/Web.xml，因为
        // String 在 java.lang 包下
        name = resolveName(name);
        // getClassLoader0 见 1.6
        ClassLoader cl = getClassLoader0();
        if (cl == null) {
            // cl 为 null，那么说明调用的这个类是一个系统中的类，
            // todo 暂时不太懂这里
            // A system class.
            return ClassLoader.getSystemResourceAsStream(name);
        }
        // 见 2.1 
        return cl.getResourceAsStream(name);
    }
```

## 1.1 Class / private String resolveName(String name)

```java
    private String resolveName(String name) {
        if (name == null) {
            return name;
        }
        // 判断 name 是否以 / 开头
        if (!name.startsWith("/")) {
            Class<?> c = this;
            // isArray 见 1.2，getComponentType 见 1.3
            // 这里 c 初始化为 this，如果 c 是数组，那么就获取数组中存储的元素的类型
            // 这里的功能其实就是递归获取数组中的类的类型
            while (c.isArray()) {
                c = c.getComponentType();
            }
            // 获取包名，详见 1.4
            String baseName = c.getName();
            // 这一步就是获取包的最后一个 . 的位置
            // 比如说 java.lang.String 就是获取 String 后面的 . 的位置，方便截取的时候
            // 截取出 java.lang
            int index = baseName.lastIndexOf('.');
            if (index != -1) {
                // 接着上面的例子，这里的 baseName.substring(0, index) 就是将 java.lang.String
                // 截取成 java.lang，然后替换成 java/lang，最后拼接上我们传进来的 name
                name = baseName.substring(0, index).replace('.', '/')
                    +"/"+name;
            }
        } else {
            // 如果以 / 开头则去掉 / 然后返回
            name = name.substring(1);
        }
        return name;
    }
```

## 1.2 Class / public native boolean isArray()

```java
    // 确定此类对象是否表示数组类
    public native boolean isArray();
```

## 1.3 Class / public native Class<?> getComponentType()

```java
    // 如果是数组类，那么则返回 componentType 
    // 其实 componentType 指的是数组中数组元素的类型，如果不是数组类型，那么 componentType 就是 null
    // 比如说: 
    //     int[].class.getComponentType() 的结果是 int
    //     int.class.getComponentType() 的结果是 null
    public native Class<?> getComponentType();
```

## 1.4 Class / public String getName()

```java
    public String getName() {
        // 这是 name 的声明 private transient String name;
        // 这个 name 其实就是类的全类名
        // 比如说：
        //    String.class.name 就是 java.lang.String 
        //    int.class.name 是 null，他不属于任何包
        String name = this.name;
        // 第一次调用这个方法的时候 name 是 null
        // 去调用 getName0 获取全类名，然后放到
        // this.name 里面缓存起来
        if (name == null)
            // getName0 见 1.5 
            this.name = name = getName0();
        return name;
    }
```

## 1.5 Class / private native String getName0()

```java
    // 这里的 getName0 就是获取类的全类名
    private native String getName0();
```

## 1.6 Class / ClassLoader getClassLoader0()

```java
    // 获取当前类的类加载器
    ClassLoader getClassLoader0() {
        return classLoader;
    }
```

# 2 public class URLClassLoader extends SecureClassLoader implements Closeable 

先看一下 URLClassLoader 的继承关系

![URLClassLoader 的继承关系](/hub/2019/December/8.png)

## 2.1 public InputStream getResourceAsStream(String name)

```java
    public InputStream getResourceAsStream(String name) {
        // 见 3.1
        // 在整个 class path 下面寻找一个叫 name 的文件
        URL url = getResource(name);
        try {
            if (url == null) {
                return null;
            }
            // 根据 url 的类型去创建一个 URLConnection 的类型
            // URLConnection 有很多种，比如 FileURLConnection、JarURLConnection 之类的
            URLConnection urlc = url.openConnection();
            // 从 urlc 获取流，到这里流就拿到了，后面就不解析了
            InputStream is = urlc.getInputStream();
            if (urlc instanceof JarURLConnection) {
                JarURLConnection juc = (JarURLConnection)urlc;
                JarFile jar = juc.getJarFile();
                synchronized (closeables) {
                    if (!closeables.containsKey(jar)) {
                        closeables.put(jar, null);
                    }
                }
            } else if (urlc instanceof sun.net.www.protocol.file.FileURLConnection) {
                synchronized (closeables) {
                    closeables.put(is, null);
                }
            }
            return is;
        } catch (IOException e) {
            return null;
        }
    }
```

# 3 public abstract class ClassLoader

## 3.1 public URL getResource(String name)

```java
    public URL getResource(String name) {
        URL url;
        // 这里便体现了双亲委派模型
        if (parent != null) {
            // 如果父加载器不为 null，那么则让父加载器去加载
            // parent 的声明 private final ClassLoader parent;
            url = parent.getResource(name);
        } else {
            // 当没有父加载器则用这个方法加载
            url = getBootstrapResource(name);
        }
        if (url == null) {
            // 其实看过了上面的 getBootstrapResource() 就会发现这里面是一样的逻辑
            // 只是如果父加载器中没有找到的话，那么子加载器也需要找找，所以就进入
            // 这个方法了，内部实现大致和 getBootstrapResource() 相同
            url = findResource(name);
        }
        return url;
    }
```

## 3.2 private static URL getBootstrapResource(String name)

```java
    private static URL getBootstrapResource(String name) {
        // 见 3.3 
        URLClassPath ucp = getBootstrapClassPath();
        // 见 5.1 
        Resource res = ucp.getResource(name);
        return res != null ? res.getURL() : null;
    }
```

## 3.3 static URLClassPath getBootstrapClassPath()

```java
    static URLClassPath getBootstrapClassPath() {
        // 见 4.1 
        return sun.misc.Launcher.getBootstrapClassPath();
    }
```

# 4 public class Launcher

## 4.1 public static URLClassPath getBootstrapClassPath()

```java
    public static URLClassPath getBootstrapClassPath() {
        // BootClassPathHolder 是 Launcher 的内部类
        // 它的定义是 private static class BootClassPathHolder
        return BootClassPathHolder.bcp;
    }
```

# 5 public class URLClassPath

## 5.1 public Resource getResource(String name)

```java
    public Resource getResource(String name) {
        // 见 5.2 
        return getResource(name, true);
    }
```

## 5.2 public Resource getResource(String name, boolean check) 

```java
    public Resource getResource(String name, boolean check) {
        if (DEBUG) {
            System.err.println("URLClassPath.getResource(\"" + name + "\")");
        }

        Loader loader;
        // 见 5.3 
        int[] cache = getLookupCache(name);
        // 见 5.4 
        // 这里所做的比较简单，其实就是找文件的加载器（Loader），然后通过加载器
        // 去判断这个文件里面比如说 jar 里面是否有我们需要的资源，如果有的话则返回
        // 没有就返回 null
        for (int i = 0; (loader = getNextLoader(cache, i)) != null; i++) {
            // 见 7.1
            Resource res = loader.getResource(name, check);
            if (res != null) {
                return res;
            }
        }
        return null;
    }
```

## 5.3 private synchronized int[] getLookupCache(String name)

```java
    private synchronized int[] getLookupCache(String name) {
        // lookupCacheURLs 的声明 private URL[] lookupCacheURLs;
        // lookupCacheEnabled 的声明 private static volatile boolean lookupCacheEnabled = "true".equals(VM.getSavedProperty("sun.cds.enableSharedLookupCache"));
        // 这里就是判断是否有缓存 url，以及缓存功能是否打开，没有缓存
        // 或者缓存功能没有打开则直接返回 null
        if (lookupCacheURLs == null || !lookupCacheEnabled) {
            return null;
        }

        int[] cache = getLookupCacheForClassLoader(lookupCacheLoader, name);
        if (cache != null && cache.length > 0) {
            int maxindex = cache[cache.length - 1]; // cache[] is strictly ascending.
            if (!ensureLoaderOpened(maxindex)) {
                if (DEBUG_LOOKUP_CACHE) {
                    System.out.println("Expanded loaders FAILED " +
                                       loaders.size() + " for maxindex=" + maxindex);
                }
                return null;
            }
        }

        return cache;
    }
```

## 5.4 private synchronized Loader getNextLoader(int[] cache, int index)

```java
    private synchronized Loader getNextLoader(int[] cache, int index) {
        if (closed) {
            return null;
        }
        if (cache != null) {
            if (index < cache.length) {
                Loader loader = loaders.get(cache[index]);
                if (DEBUG_LOOKUP_CACHE) {
                    System.out.println("HASCACHE: Loading from : " + cache[index]
                                       + " = " + loader.getBaseURL());
                }
                return loader;
            } else {
                return null; // finished iterating over cache[]
            }
        } else {
            // 见 5.5 
            return getLoader(index);
        }
    }
```

## 5.5 private synchronized Loader getLoader(int index) 

```java
     private synchronized Loader getLoader(int index) {
        if (closed) {
            return null;
        }
         // Expand URL search path until the request can be satisfied
         // or the URL stack is empty.
        while (loaders.size() < index + 1) {
            // Pop the next URL from the URL stack
            URL url;
            // urls 的声明  Stack<URL> urls = new Stack<URL>();
            // 这个存储的就是系统加载的 jar，我的系统里面存的是这个路径
            // "file:/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home/jre/classes"
            // "file:/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home/jre/lib/jfr.jar"
            // "file:/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home/jre/lib/charsets.jar"
            // "file:/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home/jre/lib/jce.jar"
            // "file:/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home/jre/lib/jsse.jar"
            // "file:/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home/jre/lib/sunrsasign.jar"
            // "file:/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home/jre/lib/rt.jar"
            // "file:/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home/jre/lib/resources.jar"
            synchronized (urls) {
                // urls 为空则返回 null，有则弹一个出来
                if (urls.empty()) {
                    return null;
                } else {
                    url = urls.pop();
                }
            }
            // Skip this URL if it already has a Loader. (Loader
            // may be null in the case where URL has not been opened
            // but is referenced by a JAR index.)
            // 见 6.1 
            String urlNoFragString = URLUtil.urlNoFragString(url);
            if (lmap.containsKey(urlNoFragString)) {
                continue;
            }
            // Otherwise, create a new Loader for the URL.
            Loader loader;
            try {
                // 见 5.6
                loader = getLoader(url);
                // If the loader defines a local class path then add the
                // URLs to the list of URLs to be opened.
                // 这里的 getClassPath 会根据 Loader 的不同而不同
                // Loader 只有两种，FileLoader 和 JarLoader
                URL[] urls = loader.getClassPath();
                if (urls != null) {
                    // 见 5.7
                    // urls 推到 this.urls 中
                    push(urls);
                }
            } catch (IOException e) {
                // Silently ignore for now...
                continue;
            } catch (SecurityException se) {
                // Always silently ignore. The context, if there is one, that
                // this URLClassPath was given during construction will never
                // have permission to access the URL.
                if (DEBUG) {
                    System.err.println("Failed to access " + url + ", " + se );
                }
                continue;
            }
            // Finally, add the Loader to the search path.
            // 见 5.8
            validateLookupCache(loaders.size(), urlNoFragString);
            // 将 loader 放入缓存中，下次查询的时候判断如果存在的话则直接跳过
            loaders.add(loader);
            lmap.put(urlNoFragString, loader);
        }
        if (DEBUG_LOOKUP_CACHE) {
            System.out.println("NOCACHE: Loading from : " + index );
        }
        // 返回结果
        return loaders.get(index);
    }
```

## 5.6 private Loader getLoader(final URL url) throws IOException

```java
    private Loader getLoader(final URL url) throws IOException {
        try {
            // 关于 java 权限相关的内容见参考文章 1 的内容
            // 这里可以简单的认为 run() 中的方法会无条件执行
            // 同时 run 中 return 的内容便是 doPrivileged return
            // 的内容
            return java.security.AccessController.doPrivileged(
                new java.security.PrivilegedExceptionAction<Loader>() {
                public Loader run() throws IOException {
                    String file = url.getFile();
                    if (file != null && file.endsWith("/")) {
                        if ("file".equals(url.getProtocol())) {
                            return new FileLoader(url);
                        } else {
                            return new Loader(url);
                        }
                    } else {
                        return new JarLoader(url, jarHandler, lmap, acc);
                    }
                }
            }, acc);
        } catch (java.security.PrivilegedActionException pae) {
            throw (IOException)pae.getException();
        }
    }
```

## 5.7 private void push(URL[] us)

```java
    private void push(URL[] us) {
        // us 推到 urls 中
        synchronized (urls) {
            for (int i = us.length - 1; i >= 0; --i) {
                urls.push(us[i]);
            }
        }
    }
```

## 5.8 private synchronized void validateLookupCache(int index, String urlNoFragString)

```java
    private synchronized void validateLookupCache(int index,
                                                  String urlNoFragString) {
        if (lookupCacheURLs != null && lookupCacheEnabled) {
            if (index < lookupCacheURLs.length &&
                urlNoFragString.equals(
                    URLUtil.urlNoFragString(lookupCacheURLs[index]))) {
                return;
            }
            if (DEBUG || DEBUG_LOOKUP_CACHE) {
                System.out.println("WARNING: resource lookup cache invalidated "
                                   + "for lookupCacheLoader at " + index);
            }
            disableAllLookupCaches();
        }
    }
```

# 6 public class URLUtil

## 6.1 public static String urlNoFragString(URL url) 

```java
    // url 转 string
    public static String urlNoFragString(URL url) {
        StringBuilder strForm = new StringBuilder();
        // 从 url 中拿出协议并拼接协议
        String protocol = url.getProtocol();
        if (protocol != null) {
            /* protocol is compared case-insensitive, so convert to lowercase */
            protocol = protocol.toLowerCase();
            strForm.append(protocol);
            strForm.append("://");
        }
        // 从 url 中拿出 host 并拼接 host
        String host = url.getHost();
        if (host != null) {
            /* host is compared case-insensitive, so convert to lowercase */
            host = host.toLowerCase();
            strForm.append(host);
            // 从 url 中拿出端口并拼接端口
            int port = url.getPort();
            if (port == -1) {
                /* if no port is specificed then use the protocols
                 * default, if there is one */
                port = url.getDefaultPort();
            }
            if (port != -1) {
                strForm.append(":").append(port);
            }
        }
        // 从 url 中拿出文件路径并拼接路径
        String file = url.getFile();
        if (file != null) {
            strForm.append(file);
        }

        return strForm.toString();
    }
```

# 7 static class JarLoader extends Loader

## 7.1 Resource getResource(final String name, boolean check)

```java
    Resource getResource(final String name, boolean check) {
        if (metaIndex != null) {
            // 见 8.1
            // metaIndex 中存放着 jar 中的所有资源
            // 如果这里判断不包含
            if (!metaIndex.mayContain(name)) {
                return null;
            }
        }

        try {
            ensureOpen();
        } catch (IOException e) {
            throw new InternalError(e);
        }
        final JarEntry entry = jar.getJarEntry(name);
        if (entry != null)
            return checkResource(name, check, entry);

        if (index == null)
            return null;

        HashSet<String> visited = new HashSet<String>();
        return getResource(name, check, visited);
    }
```

# 8 public class MetaIndex

## 8.1 public boolean mayContain(String entry)

```java
    public boolean mayContain(String entry) {
        // Ask non-class file from class only jar returns false
        // This check is important to avoid some class only jar
        // files such as rt.jar are opened for resource request.
        // 如果该 jar 包中只包含 class 文件，而我们要找的 entry
        // 不以 .class 结尾，说明该包中没有该资源
        if  (isClassOnlyJar && !entry.endsWith(".class")){
            return false;
        }
        // contents 中存放着资源文件的完整路径和包的前缀
        String[] conts = contents;
        for (int i = 0; i < conts.length; i++) {
            if (entry.startsWith(conts[i])) {
                return true;
            }
        }
        return false;
    }
```

# 参考文章

1. [Java 安全模型介绍](https://www.ibm.com/developerworks/cn/java/j-lo-javasecurity/index.html)