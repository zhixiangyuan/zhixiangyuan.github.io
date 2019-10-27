---
title: "Leetcode: 7 整数反转"
date: 2019-09-17T16:46:17+08:00
keywords: []
description: ""
tags: [
    "leetcode"
]
categories: [
]
autoCollapseToc: false
author: "yuanzx"
markup: mmark
mathjax: true  
---

# 1 题目

[整数反转](https://leetcode-cn.com/problems/reverse-integer/)

# 2 解

## 2.1 我的解

- 时间复杂度: O(n)
- 空间复杂度: O(1)

```java
class Solution {
    
    public int reverse(int num) {
        try {
            if (num >= 0) {
                String strNum = reverseInt(num);
                return Integer.parseInt(strNum);
            } else {
                String strNum = reverseInt(num * -1);
                return Integer.parseInt("-" + strNum);
            }
        } catch (NumberFormatException e) {
            // 出现异常则说明 Integer.parseInt 的时候，数溢出了
            return 0;
        }
    }
    
    private String reverseInt(int num) {
        if (num < 10) {
            return Integer.toString(num);
        }
        return Integer.toString(num % 10) + reverseInt(num / 10);
    }
}
```

