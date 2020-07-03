```json
{
  "updated_by": "KelipuTe",
  "updated_at": "2020-07-03",
  "tags": "LeetCode,算法"
}
```

---

## 118. 杨辉三角

#### 问题描述

给定一个非负整数 numRows，生成杨辉三角的前 numRows 行。

在杨辉三角中，每个数是它左上方和右上方的数的和。

示例:

```
输入: 5
输出:
[
     [1],
    [1,1],
   [1,2,1],
  [1,3,3,1],
 [1,4,6,4,1]
]
```

#### 问题分析

杨辉三角的每一行的第一个和最后一个数字都是 1。

除了前两行，后面的每一行的规律都是，下一行的第 i 个数字，等于上一行的第 i 个数字和其前一个数字的和。

#### PHP 代码实现

```php
/**
 * @param Integer $numRows
 * @return Integer[][]
 */
function generate($numRows)
{
    if ($numRows === 0) return [];

    $yh = [];
    $yh[0][0] = 1;
    for ($i = 1; $i < $numRows; $i++) {
        $yh[$i][0] = 1;
        for ($j = 1; $j < $i; $j++) {
            $yh[$i][$j] = $yh[$i - 1][$j - 1] + $yh[$i - 1][$j];
        }
        $yh[$i][$j] = 1;
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
    $resultList[] = generate($item);
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

执行用时：4 ms, 在所有 PHP 提交中击败了92.94 的用户

内存消耗：15 MB, 在所有 PHP 提交中击败了100.00% 的用户

## 参考来源

#### [118. 杨辉三角](https://leetcode-cn.com/problems/pascals-triangle/)