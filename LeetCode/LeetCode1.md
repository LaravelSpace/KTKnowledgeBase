```json
{
  "updated_by": "KelipuTe",
  "updated_at": "2020-07-03",
  "tags": "LeetCode,算法"
}
```

---

## 1. 两数之和

#### 问题描述

给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。

你可以假设每种输入只会对应一个答案。但是，数组中同一个元素不能使用两遍。

示例:

```
给定 nums = [2, 7, 11, 15], target = 9

因为 nums[0] + nums[1] = 2 + 7 = 9
所以返回 [0, 1]
```

#### 问题分析

暴力循环是可以求解的，时间复杂度为 O(n^2)。这里不用暴力循环。

遍历原数组，构造以原数组的键值为键名，原数组的键名为键值的 map。

遍历原数组，用目标值减去数组键值，得到另一个目标数的数值。

判断另一个目标数在不在之前构造的 map 里。

这里需要注意的是，需要额外增加判断，处理原数组里数字重复出现两次的情况。

#### PHP 代码实现

```php
/**
 * @param Integer[] $nums
 * @param Integer $target
 * @return Integer[]
 */
function twoSum($nums, $target)
{
    $numberMap = [];
    // 遍历第一遍构造 map
    foreach ($nums as $index => $value) {
        $numberMap[(string)$value] = $index;
    }
    // 遍历第二遍找答案
    foreach ($nums as $index => $value) {
        $otherValue = $target - $value;
        $otherKey = (string)$otherValue;
        if (isset($numberMap[$otherKey]) && $index !== $numberMap[$otherKey]) {
            return [$index, $numberMap[$otherKey]];
        }
    }
    return [];
}
```

测试代码：

```php
$testList = [
    ['nums' => [3, 2, 4], 'target' => 6],
    ['nums' => [5, 3, 3, 2], 'target' => 6],
    ['nums' => [2, 7, 11, 15], 'target' => 9],
];
$resultList = [];

$timeStart = intval(microtime(true) * 1000);
foreach ($testList as $item) {
    $resultList[] = twoSum($item['nums'], $item['target']);
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

执行用时：16 ms, 在所有 PHP 提交中击败了80.81% 的用户

内存消耗：16.1 MB, 在所有 PHP 提交中击败了100.00% 的用户

## 参考来源

#### [1. 两数之和](https://leetcode-cn.com/problems/two-sum/)