---
title: "sed 学习日记"
date: 2019-07-04T16:21:59+08:00
keywords: []
description: ""
tags: [
    "Linux"
]
categories: [
    "杂货铺"
]
author: "yuanzx"
---

# 一、sed 的工作流程

sed 逐行处理文件或输入，默认不会修改文件，除非使用 shell 重定向保存结果。

工作流程：

1. 将正在处理的行保存在一个临时缓存区中（也称为模式空间，还有个保持空间，这两个什么关系？）
2. 处理缓冲区中的行
3. 处理完之后将其发送到屏幕上并将其从缓冲区中删除
4. 读取下一行
5. 如此循环到最后一行然后结束

# 二、命令参数

```shell
sed [-e] 'command' [-f] scriptfile files

-e 参数后面直接加命令，可以指定多次
-f 参数后面使用脚本文件，可以指定多次
files 是需要处理的文件，或者以管道的方式输入数据
```

# 三、实际示例

## 使用 sed 达到的 cat 的效果

```shell
$>cat testfile

There is only one thing that makes a dream impossible to achieve: the fear of failure. 
 - Paulo Coelho, The Alchemist

$>sed '' testfile

There is only one thing that makes a dream impossible to achieve: the fear of failure. 
 - Paulo Coelho, The Alchemist
```

## 删除第一行第三行的文本

```shell
$>cat testfile

1) A Storm of Swords, George R. R. Martin, 1216 
2) The Two Towers, J. R. R. Tolkien, 352 
3) The Alchemist, Paulo Coelho, 197 
4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
5) The Pilgrimage, Paulo Coelho, 288 
6) A Game of Thrones, George R. R. Martin, 864

$>sed -e '1d' -e '2d' testfile

3) The Alchemist, Paulo Coelho, 197
4) The Fellowship of the Ring, J. R. R. Tolkien, 432
5) The Pilgrimage, Paulo Coelho, 288 
6) A Game of Thrones, George R. R. Martin, 864
```

## 将命令写在文本文件中

```shell
$>cat testfile

1) A Storm of Swords, George R. R. Martin, 1216 
2) The Two Towers, J. R. R. Tolkien, 352 
3) The Alchemist, Paulo Coelho, 197 
4) The Fellowship of the Ring, J. R. R. Tolkien, 432 

$>cat scriptfile

1d
3d

$>sed -f scriptfile testfile

2) The Two Towers, J. R. R. Tolkien, 352 
4) The Fellowship of the Ring, J. R. R. Tolkien, 432
```

## 

## 参考资料

1. [Linux 命令大全](http://man.linuxde.net/sed)
2. [三十分钟学会 SED](https://www.jianshu.com/p/303618e3e1db?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation)
3. [linux sed 超强使用指南](https://www.jianshu.com/p/2e13d84456c6)
4. [sed, a stream editor](https://www.gnu.org/software/sed/manual/sed.html)



