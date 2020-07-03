```json
{
  "updated_by": "KelipuTe",
  "updated_at": "2020-07-03",
  "tags": "LeetCode,算法,二叉树"
}
```

---

## 107.二叉树的层次遍历 II

#### 问题描述

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

#### 问题分析

使用广度优先(层次)遍历，分层记录每一层的遍历结果。结果的层次顺序是反的，最后需要把数组倒过来。

#### PHP 代码实现

ErChaShu.php 见 ErChaShu.md 的内容。

```php
/**
 * Definition for a binary tree node.
 * class TreeNode {
 *     public $val = null;
 *     public $left = null;
 *     public $right = null;
 *     function __construct($value) { $this->val = $value; }
 * }
 */

require_once 'ErChaShu.php';

/**
 * @param TreeNode $root
 * @return Integer[][]
 */
function levelOrderBottom($root)
{
    $nodeList = [];
    $level = 1;

    $queue = [];
    array_push($queue, $root);
    while ($length = count($queue)) {
        for ($i = 0; $i < $length; $i++) {
            $tNode = array_shift($queue);
            if ($tNode->val !== null) $nodeList[$level][] = $tNode->val;
            if ($tNode->left !== null) array_push($queue, $tNode->left);
            if ($tNode->right !== null) array_push($queue, $tNode->right);
        }
        $level++;
    }

    return array_reverse($nodeList);
}
```

测试代码：

```php
$numList = [3, 9, 20, null, null, 15, 7];
$numTree = makeTree($numList);
$testList = [$numTree];
$resultList = [];

$timeStart = intval(microtime(true) * 1000);
foreach ($testList as $item) {
    $resultList[] = levelOrderBottom($item);
}
$timeStop = intval(microtime(true) * 1000);

$echo = [
    'timeStart' => $timeStart,
    'timeStop' => $timeStop,
    'timePass' => $timeStop - $timeStart,
    'result' => $resultList
];
echo json_encode($echo);
```

#### 测试结果

执行结果：通过

执行用时：12 ms, 在所有 PHP 提交中击败了40.00% 的用户

内存消耗：15.3 MB, 在所有 PHP 提交中击败了100.00% 的用户

## 参考来源

#### [107. 二叉树的层次遍历 II](https://leetcode-cn.com/problems/binary-tree-level-order-traversal-ii/)