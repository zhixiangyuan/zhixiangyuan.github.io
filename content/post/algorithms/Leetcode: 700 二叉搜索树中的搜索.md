---
title: "Leetcode: 700 二叉搜索树中的搜索"
date: 2019-09-29T08:01:03+08:00
keywords: []
description: ""
tags: [
    "leetcode"
]
categories: [
]
autoCollapseToc: false
author: "yuanzx"
---

# 1 题目

[二叉搜索树中的搜索](https://leetcode-cn.com/problems/binary-watch/)

# 2 解

## 2.1 我的解

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public TreeNode searchBST(TreeNode root, int val) {
        if (root == null) return null;
        if (root.val == val) return root;
        if (root.val < val) return searchBST(root.right, val);
        else return searchBST(root.left, val);
    }
}
```