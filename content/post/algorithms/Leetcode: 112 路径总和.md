---
title: "Leetcode: 112 路径总和"
date: 2019-09-18T06:55:44+08:00
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
markup: mmark
mathjax: true 
---

# 1 题目

[路径总和](https://leetcode-cn.com/problems/path-sum/)

# 2 解

## 2.1 我的解

- 空间复杂度: O(log(n))
- 时间复杂度: O(n)

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
    public boolean hasPathSum(TreeNode root, int sum) {
        if (root == null) return false;
        return hasPathSum(root, sum, 0);
    }
    
    private boolean hasPathSum(TreeNode root, int sum, int calSum) {
        if (root.left == null && root.right == null 
            && sum == calSum + root.val) return true;
        if (root.left != null &&
            hasPathSum(root.left, sum, calSum + root.val)) return true;
        if (root.right != null &&
            hasPathSum(root.right, sum, calSum + root.val)) return true;
        return false;
    }
}
```

## 2.2 官方的解

### 2.2.1 递归

和我的稍有不同，官方采用减法，达到叶子结点的时候比较值是否为 0，很巧妙。

- 时间复杂度：我们访问每个节点一次，时间复杂度为 O(N) ，其中 N 是节点个数。
- 空间复杂度：最坏情况下，整棵树是非平衡的，例如每个节点都只有一个孩子，递归会调用 N 次（树的高度），因此栈的空间开销是 O(N) 。但在最好情况下，树是完全平衡的，高度只有 log(N)，因此在这种情况下空间复杂度只有 O(log(N)) 。

```java
class Solution {
  public boolean hasPathSum(TreeNode root, int sum) {
    if (root == null)
      return false;

    sum -= root.val;
    if ((root.left == null) && (root.right == null))
      return (sum == 0);
    return hasPathSum(root.left, sum) || hasPathSum(root.right, sum);
  }
}
```

### 2.2.2 迭代

迭代的方式也是很巧妙，在之前的学习中，只想到过用队列来做迭代，可以实现层序遍历的效果，而这里通过栈做遍历，实现了遍历某一条子树的效果，通过使用两个栈保证原节点的 val 不被修改。

- 时间复杂度: O(n)
- 空间复杂度: 当树不平衡的最坏情况下是 O(n), 最好情况下（树是平衡的）是 O(log n)

```java
class Solution {
  public boolean hasPathSum(TreeNode root, int sum) {
    if (root == null)
      return false;

    LinkedList<TreeNode> node_stack = new LinkedList();
    LinkedList<Integer> sum_stack = new LinkedList();
    node_stack.add(root);
    sum_stack.add(sum - root.val);

    TreeNode node;
    int curr_sum;
    while ( !node_stack.isEmpty() ) {
      node = node_stack.pollLast();
      curr_sum = sum_stack.pollLast();
      if ((node.right == null) && (node.left == null) && (curr_sum == 0))
        return true;

      if (node.right != null) {
        node_stack.add(node.right);
        sum_stack.add(curr_sum - node.right.val);
      }
      if (node.left != null) {
        node_stack.add(node.left);
        sum_stack.add(curr_sum - node.left.val);
      }
    }
    return false;
  }
}
```