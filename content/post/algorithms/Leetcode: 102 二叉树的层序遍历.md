---
title: "Leetcode: 102 二叉树的层序遍历"
date: 2019-09-12T08:08:15+08:00
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

[二叉树的前序遍历](https://leetcode-cn.com/problems/binary-tree-preorder-traversal/)

# 2 解

## 2.1 我的解

由于他要对每一层进行分层，所以我想的就是通过 `List<List<TreeNode>>` 去分层，但是我这个解跑出来的效果看上去时间复杂度和空间复杂度都不好，无语。

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
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> result = new ArrayList<>();
        List<List<TreeNode>> nodeList = new ArrayList<>();
        if (root == null) return result;
        result.add(new ArrayList(){{add(root.val);}});
        nodeList.add(new ArrayList(){{add(root);}});
        nodeList.add(new ArrayList(){{
            if (root.left != null) add(root.left); 
            if (root.right != null) add(root.right);
        }});
        recursive(result, nodeList, 1);
        return result;
    }
    
    private void recursive(List<List<Integer>> result, 
                           List<List<TreeNode>> nodeList, int layer){
        if (layer + 1> nodeList.size()) return;
        List<TreeNode> subNodeList = nodeList.get(layer);
        List<Integer> numList = new ArrayList<>();
        List<TreeNode> nextLayerNodeList = new ArrayList<>();
        for (TreeNode node : subNodeList) {
            numList.add(node.val);
            if (node.left != null) nextLayerNodeList.add(node.left);
            if (node.right != null) nextLayerNodeList.add(node.right);
        }
        if (numList.size() > 0) result.add(numList);
        if (nextLayerNodeList.size() > 0) nodeList.add(nextLayerNodeList);
        recursive(result, nodeList, layer + 1);
    }
}
```