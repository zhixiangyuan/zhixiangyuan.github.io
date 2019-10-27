---
title: "Leetcode: 98 验证二叉搜索树"
date: 2019-09-24T07:09:35+08:00
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

[验证二叉搜索树](https://leetcode-cn.com/problems/validate-binary-search-tree/)

# 2 解

## 2.1 我的解

- 时间复杂度: O(n)
- 空间复杂度: O(n)

本解中的 list 可以换成 stack，只存储上一个节点数，这样就能节省空间。

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
    public boolean isValidBST(TreeNode root) {
        if (root == null) return true;
        ArrayList<Integer> list = new ArrayList<>();
        return isValidBST(root, list);
    }
    
    private boolean isValidBST(TreeNode root, List<Integer> list) {
        if (root == null) return true;
        if (!isValidBST(root.left, list)) return false;
        if (list.size() > 0 
            && list.get(list.size() - 1) >= root.val) {
            return false;
        } else {
            list.add(root.val);
        }
        if (!isValidBST(root.right, list)) return false;
        return true;
    }
}
```