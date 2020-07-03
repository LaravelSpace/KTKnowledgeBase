```json
{
  "updated_by": "KelipuTe",
  "updated_at": "2020-07-03",
  "tags": "LeetCode,算法"
}
```

---

## 119. 杨辉三角 II

#### 问题描述

给定一个非负索引 k，其中 k ≤ 33，返回杨辉三角的第 k 行。

在杨辉三角中，每个数是它左上方和右上方的数的和。

示例:

```
输入: 3
输出: [1,3,3,1]
```

进阶：

你可以优化你的算法到 O(k) 空间复杂度吗？

#### 问题分析

杨辉三角的每一行的第一个和最后一个数字都是 1。

除了前两行，后面的每一行的规律都是，下一行的第 i 个数字，等于上一行的第 i 个数字和其前一个数字的和。

这题额外的要求是使用一维数组求解，所以在计算下一行时，可以利用多出来的一个位置，从后往前算。

比如计算 [1,3,3,1] 到 [1,4,6,4,1] 时，倒着算先算最后一位 [1,3,3,1,1]，

然后倒数第二位 4 等于倒数第二位与倒数第三位的和 [1,3,3,4,1]，依次类推。

这样可以保证在计算下一行时，所依赖的上一行的结果，不会被新的数字替换掉，顺序计算时会遇到这个问题。

#### PHP 代码实现

```php
/**
 * @param Integer $rowIndex
 * @return Integer[]
 */
function getRow($rowIndex)
{
    $rowIndex += 1;
    if ($rowIndex === 0) return [];

    $yh = [];
    $yh[0] = 1;
    for ($i = 1; $i < $rowIndex; $i++) {
        $yh[$i] = 1;
        for ($j = $i - 1; $j > 0; $j--) {
            $yh[$j] = $yh[$j] + $yh[$j - 1];
        }
    }
    
    return $yh;
}
```

测试代码：

```php
$testList = [5, 6];
$resultList = [];

$timeStart = intval(microtime(true) * 1000);
foreach ($testList as $item) {
    $resultList[] = getRow($item);
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

执行用时：0 ms, 在所有 PHP 提交中击败了100.00% 的用户

内存消耗：15 MB, 在所有 PHP 提交中击败了100.00% 的用户

## 参考来源

#### [119. 杨辉三角 II](https://leetcode-cn.com/problems/pascals-triangle-ii/)