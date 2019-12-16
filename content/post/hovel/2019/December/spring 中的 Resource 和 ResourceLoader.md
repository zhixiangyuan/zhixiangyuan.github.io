---
title: "spring 中的 Resource 和 ResourceLoader"
date: 2019-12-16T10:00:29+08:00
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

# 1 Resource

由于 Java 定义的 java.net.URL（Uniform Resource Locator）统一资源定位器有些狭隘，无法做到统一定位资源的作用（todo 这里有一个疑问，哪些资源定位不到呢？），所以 Spring core 内部实现了 org.springframework.core.io.Resource 来定位所有的资源，先看一下 Resource 的内部方法。

```java
public interface InputStreamSource {

	/** 获取 InputStream 流 */
	InputStream getInputStream() throws IOException;

}

public interface Resource extends InputStreamSource {

	/** 判断资源是否存在 */
	boolean exists();

	/** 资源是否可读 */
	default boolean isReadable() {
		return exists();
	}

	/** 资源是否打开，简单来讲就是是否可以获取 InputStream 流 */
	default boolean isOpen() {
		return false;
	}

	/** 资源是否是文件系统中的文件 */
	default boolean isFile() {
		return false;
	}

	/** 返回一个 URL */
	URL getURL() throws IOException;

	/** 返回一个 URI */
	URI getURI() throws IOException;

	/** 返回一个 File */
	File getFile() throws IOException;

	/** 将流封装成 java.nio.channels.Channel 进行返回*/
	default ReadableByteChannel readableChannel() throws IOException {
		return Channels.newChannel(getInputStream());
	}

	/** 返回内容的长度，应该是字节长度 */
	long contentLength() throws IOException;

	/** 返回最后修改的 Unix 时间戳 */
	long lastModified() throws IOException;

	/** 根据路径创建资源对象 */
	Resource createRelative(String relativePath) throws IOException;

	/** 返回资源名 */
	@Nullable
	String getFilename();

	/** 返回资源描述 */
	String getDescription();

}
```

下图 UML 展示一些常见的 Resource 实现

![](/hub/2019/December/21.png)

ByteArrayResource: 为字节数组类型的资源提供封装，将其构造成 ByteArrayInputStream 进行返回，供调用者使用。

ClassPathResource: 该实现从 Java 应用程序的 ClassPath 中加载具体资源并进行封装，可以使用指定的类加载器或者给定的类进行资源加载。

FileSystemResource: 对 java.io.File 类型的资源进行封装，关于 File 类型的资源都可以使用这个类加载。

UrlResource: 通过 java.net.URL 进行具体资源查找定位的实现类，内部委派给 URL 进行具体的资源操作。

InputStreamResource: InputStream 类型的资源使用这个类进行加载。

AbstractResource: 对 Resource 的抽象实现，当我们需要自己实现相关的资源类的时候可以和 Spring 一样，直接继承这个抽象类实现我们关心的方法即可。

# 2 ResourceLoader

Resource 负责定位资源，ResourceLoader 负责加载资源，还有一个和 ResourceLoader 相关的 ResourcePatternResolver，它负责根据 Ant 风格的匹配模式去批量加载资源。

```java
public interface ResourceLoader {

	/** ResourceUtils.CLASSPATH_URL_PREFIX = "classpath:" */
	String CLASSPATH_URL_PREFIX = ResourceUtils.CLASSPATH_URL_PREFIX;


	/** 从指定的位置加载资源 */
	Resource getResource(String location);

	/** 返回加载资源使用的 ClassLoader */
	@Nullable
	ClassLoader getClassLoader();

}

public interface ResourcePatternResolver extends ResourceLoader {

	String CLASSPATH_ALL_URL_PREFIX = "classpath*:";

	/** 根据 Ant 风格的匹配模式去加载资源 */
	Resource[] getResources(String locationPattern) throws IOException;

}
```

下图 UML 展示常见的 ResourceLoader 和 ResourcePatternResolver 实现

![](/hub/2019/December/22.png)

DefaultResourceLoader: 这个类是对于 ResourceLoader 的默认实现。

FileSystemResourceLoader: 由于 DefaultResourceLoader 对于文件的加载不够完美，所以 FileSystemResourceLoader 应运而生，内部覆写了 getResourceByPath 方法。

PathMatchingResourcePatternResolver: 是 ResourcePatternResolver 的默认实现，实现批量加载资源的功能，内部的 resourceLoader 字段默认使用 DefaultResourceLoader，也可以替换成别的类。

# 参考文章

1. 《Spring 揭秘》