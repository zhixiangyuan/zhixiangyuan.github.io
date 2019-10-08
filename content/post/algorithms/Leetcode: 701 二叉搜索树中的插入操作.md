---
title: "Leetcode: 701 二叉搜索树中的插入操作"
date: 2019-10-08T08:05:16+08:00
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

[二叉搜索树中的插入操作](https://leetcode-cn.com/problems/insert-into-a-binary-search-tree/submissions/)

# 2 解

## 2.1 我的解

- 时间复杂度: O(logn)
- 空间复杂度: O(1)

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
    public TreeNode insertIntoBST(TreeNode root, int val) {
        if (root == null) {
            return new TreeNode(val);
        }
        if (root.val > val) {
            if (root.left == null) {
                TreeNode entry = new TreeNode(val);
                root.left = entry;
            } else {
                insertIntoBST(root.left, val);
            }
        } else if (root.val < val) {
            if (root.right == null) {
                TreeNode entry = new TreeNode(val);
                root.right = entry;
            } else {
                insertIntoBST(root.right, val);
            }
        }
        return root;
    }
}
```