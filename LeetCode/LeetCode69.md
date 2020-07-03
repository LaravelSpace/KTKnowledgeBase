```json
{
  "updated_by": "KelipuTe",
  "updated_at": "2020-07-03",
  "tags": "LeetCode,算法,二分查找"
}
```

---

## 69. x 的平方根

#### 问题描述

实现 int sqrt(int x) 函数。

计算并返回 x 的平方根，其中 x 是非负整数。

由于返回类型是整数，结果只保留整数的部分，小数部分将被舍去。

示例 1:

```
输入: 4
输出: 2
```

示例 2:

```
输入: 8
输出: 2
说明: 8 的平方根是 2.82842..., 
     由于返回类型是整数，小数部分将被舍去。
```

#### 问题分析

由于这题只需要计算到整数部分，相当于在数列 [1,4,9,16,25,36,...] 中查找一个数字的位置。

使用二分查询，区别在于，可能查不到目标数值。

这个时候每次查询结束时，需要额外进行一次比较，确认目标是否正好卡在端点前后。

#### PHP 代码实现

```php
/**
 * @param Integer $x
 * @return Integer
 */
function mySqrt($x)
{
    $left = 0;
    $right = intval($x / 2) + 1;
    while ($left <= $right) {
        $middle = intval(($left + $right) / 2);
        if ($middle * $middle < $x) {
            if (($middle + 1) * ($middle + 1) > $x) return $middle;
            $left = $middle + 1;
        } else if ($middle * $middle > $x) {
            if (($middle - 1) * ($middle - 1) < $x) return $middle - 1;
            $right = $middle - 1;
        } else {
            return $middle;
        }
    }
}
```

测试代码：

```php
$testList = [0, 1, 2, 4, 8, 15, 18, 35];
$resultList = [];

$timeStart = intval(microtime(true) * 1000);
foreach ($testList as $item) {
    $resultList[] = mySqrt($item);
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

执行用时：12 ms, 在所有 PHP 提交中击败了68.20% 的用户

内存消耗：15 MB, 在所有 PHP 提交中击败了100.00% 的用户

## 参考来源

#### [69. x 的平方根](https://leetcode-cn.com/problems/sqrtx/)