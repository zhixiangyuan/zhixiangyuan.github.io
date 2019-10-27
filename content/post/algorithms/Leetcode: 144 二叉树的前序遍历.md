---
title: "Leetcode: 144 二叉树的前序遍历"
date: 2019-09-12T07:09:30+08:00
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

[二叉树的前序遍历](https://leetcode-cn.com/problems/binary-tree-preorder-traversal/)

# 2 解

## 2.1 我的解

- 时间复杂度 O(n)
- 空间复杂度 O(n)

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
    public List<Integer> preorderTraversal(TreeNode root) {
        List<Integer> result = new ArrayList<>();
        recursive(root, result);
        return result;
    }
    
    private void recursive(TreeNode root, List<Integer> result) {
        if (root == null) return;
        result.add(root.val);
        recursive(root.left, result);
        recursive(root.right, result);
    }
}
```