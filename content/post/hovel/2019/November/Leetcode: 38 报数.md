---
title: "Leetcode: 38 报数"
date: 2019-11-11T08:22:38+08:00
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

[报数](https://leetcode-cn.com/problems/count-and-say/submissions/)

# 2 解

```java
class Solution {
    /** 编译正则，按照重复的数字分组 */
    private Pattern pattern = Pattern.compile("(\\d)\\1*");
    public String countAndSay(int n) {
        if (n == 1) return "1";
        StringBuilder result = new StringBuilder();
        result.append("1");
        for (int i = 1; i < n; i++){
            Matcher matcher = pattern.matcher(result);
            result = new StringBuilder();
            // 找出数字相同的分组，然后循环
            while(matcher.find()) {
                String group = matcher.group();
                char[] chars = group.toCharArray();
                // 先读数组长度，再读数组第一个数字即可
                result.append(chars.length).append(chars[0]);
            }
        }
        return result.toString();
    }
}
```