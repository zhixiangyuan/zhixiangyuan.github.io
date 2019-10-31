---
title: "Java Gzip 的简单封装"
date: 2019-07-04T14:50:20+08:00
keywords: []
description: ""
tags: [
    "Java"
]
categories: [
    "杂货铺"
]
author: "yuanzx"
---

>本文基于 Jdk 1.8

```java
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.util.zip.GZIPInputStream;
import java.util.zip.GZIPOutputStream;

public class GzipUtil {

    /**
     * @param bytes
     * @return 压缩
     */
    public static byte[] compress(byte[] bytes) throws IOException {
        if (bytes == null || bytes.length == 0) {
            return bytes;
        }
        try (
                ByteArrayOutputStream out = new ByteArrayOutputStream();
                GZIPOutputStream gzip = new GZIPOutputStream(out);
        ) {
            gzip.write(bytes);
            gzip.finish();
            return out.toByteArray();
        }
    }

    /**
     * @param bytes
     * @return 解压缩
     */
    public static byte[] uncompress(byte[] bytes) throws IOException {
        if (bytes == null || bytes.length == 0) {
            return bytes;
        }
        try (
                ByteArrayOutputStream out = new ByteArrayOutputStream();
                ByteArrayInputStream in = new ByteArrayInputStream(bytes);
                GZIPInputStream gzip = new GZIPInputStream(in);
        ) {
            byte[] buffer = new byte[1024];
            int offset;
            while ((offset = gzip.read(buffer)) != -1) {
                out.write(buffer, 0, offset);
            }
            return out.toByteArray();
        }
    }
}
```