```json
{
  "updated_by": "KelipuTe",
  "updated_at": "2020-07-03",
  "tags": "LeetCode,算法,二叉树"
}
```

---

## 199. 二叉树的右视图

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

#### 问题分析

使用广度优先(层次)遍历，分层记录每一层的遍历结果，找到每一层最右侧的元素。

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
 * @return Integer[]
 */
function rightSideView($root)
{
    $nodeList = [];
    
    if ($root === null || $root->val === null) return [];

    $queue = [];
    array_push($queue, $root);
    while ($length = count($queue)) {
        $tempResult = [];
        for ($i = 0; $i < $length; $i++) {
            $tNode = array_shift($queue);
            // 用右侧的节点覆盖左侧的节点
            if ($tNode->val !== null) $tempResult = $tNode->val;
            if ($tNode->left !== null) array_push($queue, $tNode->left);
            if ($tNode->right !== null) array_push($queue, $tNode->right);

        }
        $nodeList[] = $tempResult;
    }
    
    return $nodeList;
}
```

测试代码：

```php
$numList = [1, 2, 3, null, 5, null, 4];
$numTree = makeTree($numList);
$testList = [$numTree];
$resultList = [];

$timeStart = intval(microtime(true) * 1000);
foreach ($testList as $item) {
    $resultList[] = rightSideView($item);
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

执行用时：4 ms, 在所有 PHP 提交中击败了89.32% 的用户

内存消耗：14.8 MB, 在所有 PHP 提交中击败了100.00% 的用户

## 参考来源

#### [199. 二叉树的右视图](https://leetcode-cn.com/problems/binary-tree-right-side-view/)