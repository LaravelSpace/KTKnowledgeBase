```json
{
  "updated_by": "KelipuTe",
  "updated_at": "2020-07-03",
  "tags": "LeetCode,算法,链表"
}
```

---

## 2. 两数相加

#### 问题描述

给出两个 非空 的链表用来表示两个非负的整数。其中，它们各自的位数是按照 逆序 的方式存储的，并且它们的每个节点只能存储 一位 数字。

如果，我们将这两个数相加起来，则会返回一个新的链表来表示它们的和。

您可以假设除了数字 0 之外，这两个数都不会以 0 开头。

示例：

```
输入：(2 -> 4 -> 3) + (5 -> 6 -> 4)
输出：7 -> 0 -> 8
原因：342 + 465 = 807
```

#### 问题分析

注意两个数字链表可能不一样长，以及进位时的情况。

#### PHP 代码实现

LianBiao.php 见 LianBiao.md 的内容。

```php
/**
 * Definition for a singly-linked list.
 * class ListNode {
 *     public $val = 0;
 *     public $next = null;
 *     function __construct($val) { $this->val = $val; }
 * }
 */

require_once 'LianBiao.php';

/**
 * @param ListNode $l1
 * @param ListNode $l2
 * @return ListNode
 */
function addTwoNumbers($l1, $l2)
{
    $listHead = null;
    $listTail = null;
    $nextCarry = false;
    $l1Node = $l1;
    $l2Node = $l2;
    do {
        $prevCarry = $nextCarry ? 1 : 0;
        $l1Val = ($l1Node !== null) ? $l1Node->val : 0;
        $l2Val = ($l2Node !== null) ? $l2Node->val : 0;
        $thisVal = $prevCarry + $l1Val + $l2Val;

        $nextCarry = false;
        if ($thisVal >= 10) {
            $nextCarry = true;
            $thisVal -= 10;
        }

        $listNode = new ListNode($thisVal);
        if ($listHead === null) {
            $listHead = $listNode;
            $listTail = $listNode;
        } else {
            $listTail->next = $listNode;
            $listTail = $listNode;
        }
        if ($l1Node !== null) $l1Node = $l1Node->next;
        if ($l2Node !== null) $l2Node = $l2Node->next;
    } while ($l1Node !== null || $l2Node !== null || $nextCarry);

    return $listHead;
}
```

测试代码：

```php
$testList = [
    ['num1' => 342, 'num2' => 46588],
    ['num1' => 5, 'num2' => 5],
    ['num1' => 0, 'num2' => 10],
];
foreach ($testList as $index => $item) {
    $testList[$index]['num1'] = makeListAsc($item['num1']);
    $testList[$index]['num2'] = makeListAsc($item['num2']);
}
$resultList = [];

$timeStart = intval(microtime(true) * 1000);
foreach ($testList as $item) {
    $resultList[] = addTwoNumbers($item['num1'], $item['num2']);
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

执行用时：20 ms, 在所有 PHP 提交中击败了85.91 的用户

内存消耗：14.9 MB, 在所有 PHP 提交中击败了100.00% 的用户

## 参考来源

#### [2. 两数相加](https://leetcode-cn.com/problems/add-two-numbers/)