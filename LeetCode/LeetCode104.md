```json
{
  "updated_by": "KelipuTe",
  "updated_at": "2020-07-03",
  "tags": "LeetCode,算法,二叉树"
}
```

---

## 104. 二叉树的最大深度

#### 问题描述

给定一个二叉树，找出其最大深度。二叉树的深度为根节点到最远叶子节点的最长路径上的节点数。

说明: 叶子节点是指没有子节点的节点。

示例：给定二叉树 [3,9,20,null,null,15,7]

```
    3
   / \
  9  20
    /  \
   15   7
```

返回它的最大深度 3 。

#### 问题分析

使用深度优先遍历，记录每个节点的深度，找到最大深度

#### PHP 代码实现

```php
/**
 * 递归解法
 * @param TreeNode $root
 * @return Integer
 */
function maxDepth($root)
{
    if ($root === null) return 0;
    $left = maxDepth($root->left);
    $right = maxDepth($root->right);
    return max($left, $right) + 1;
}
```

#### 测试代码

```php
$numList = [3, 9, 20, null, null, 15, 7];
$numTree = makeTree($numList);
$testList = [$numTree];
$resultList = [];

$timeStart = intval(microtime(true) * 1000);
foreach ($testList as $item) {
    $resultList[] = maxDepth($item);
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

执行用时：12 ms, 在所有 PHP 提交中击败了80.37% 的用户

内存消耗：16.2 MB, 在所有 PHP 提交中击败了100.00% 的用户

## 参考来源

#### [104. 二叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/)