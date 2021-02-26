## LeetCode151-200题

### [199. 二叉树的右视图](https://leetcode-cn.com/problems/binary-tree-right-side-view/)

#### 问题描述

给定一棵二叉树，想象自己站在它的右侧，按照从顶部到底部的顺序，返回从右侧所能看到的节点值。

示例:

```
输入: [1,2,3,null,5,null,4]
输出: [1, 3, 4]
解释:
   1            <---
 /   \
2     3         <---
 \     \
  5     4       <---

```

问题分析：使用广度优先(层次)遍历，分层记录每一层的遍历结果，找到每一层最右侧的元素。

代码参考：CYangLi/leetcode/leetcode199.c