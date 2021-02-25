## LeetCode101-150题

### [104. 二叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/)

给定一个二叉树，找出其最大深度。二叉树的深度为根节点到最远叶子节点的最长路径上的节点数。

说明：叶子节点是指没有子节点的节点。

示例：给定二叉树 [3,9,20,null,null,15,7]，返回它的最大深度 3 。

     3
    / \
    9  20
      /  \
     15   7

问题分析：用队列的方式进行层次遍历。

代码参考：CYangLi/leetcode/leetcode104.c

### [107. 二叉树的层序遍历 II](https://leetcode-cn.com/problems/binary-tree-level-order-traversal-ii/)

给定一个二叉树，返回其节点值自底向上的层次遍历。 （即按从叶子节点所在层到根节点所在的层，逐层从左向右遍历）

例如：给定二叉树 [3,9,20,null,null,15,7],

```
    3
   / \
  9  20
    /  \
   15   7
```

返回其自底向上的层次遍历为：

```
[
  [15,7],
  [9,20],
  [3]
]
```

问题分析：使用广度优先(层次)遍历，分层记录每一层的遍历结果。结果的层次顺序是反的，最后需要把数组倒过来。

代码参考：CYangLi/leetcode/leetcode107.c

### [108. 将有序数组转换为二叉搜索树](https://leetcode-cn.com/problems/convert-sorted-array-to-binary-search-tree/)

给你一个整数数组nums，其中元素已经按升序排列，请你将其转换为一棵高度平衡二叉搜索树。高度平衡二叉树是一棵满足「每个节点的左右两个子树的高度差的绝对值不超过1」的二叉树。

示例：

```
输入：nums = [-10,-3,0,5,9]
输出：[0,-3,9,-10,null,5]
解释：[0,-10,5,null,-3,null,9] 也将被视为正确答案
```

提示：

    1 <= nums.length <= 104
    -104 <= nums[i] <= 104
    nums 按 严格递增 顺序排列

问题分析：数组已经有序了，那么从有序数组从中间一分为二，左边的一半都比中间值小，右边的一半都比中间值大。然后递归操作处理每一段。

代码参考：CYangLi/leetcode/leetcode108.c