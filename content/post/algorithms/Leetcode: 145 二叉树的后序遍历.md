---
title: "Leetcode: 145 二叉树的后序遍历"
date: 2019-09-12T07:21:13+08:00
keywords: []
description: ""
tags: [
    "leetcode"
]
categories: [
    "数据结构与算法"
]
autoCollapseToc: false
author: "yuanzx"
---

# 1 题目

[二叉树的后序遍历](https://leetcode-cn.com/problems/binary-tree-postorder-traversal/)

# 2 解

## 2.1 我的解

- 时间复杂度 O(n) // 这里应该是 O(n) 吧，毕竟 n 个节点就要遍历 n 次
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
    public List<Integer> postorderTraversal(TreeNode root) {
        List<Integer> result = new ArrayList<>();
        recursive(root, result);
        return result;
    }
    
    private void recursive(TreeNode root, List<Integer> result) {
        if (root == null) return;
        recursive(root.left, result);
        recursive(root.right, result);
        result.add(root.val);
    }
}
```