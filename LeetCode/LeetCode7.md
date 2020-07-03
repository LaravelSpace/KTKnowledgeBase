```json
{
  "updated_by": "KelipuTe",
  "updated_at": "2020-07-03",
  "tags": "LeetCode,算法"
}
```

---

## 7. 整数反转

#### 问题描述

给出一个 32 位的有符号整数，你需要将这个整数中每位上的数字进行反转。

示例 1:

```
输入: 123
输出: 321
```

 示例 2:

```
输入: -123
输出: -321
```

示例 3:

```
输入: 120
输出: 21
```

注意:

假设我们的环境只能存储得下 32 位的有符号整数，则其数值范围为 [−231,  231 − 1]。请根据这个假设，如果反转后整数溢出那么就返回 0。

#### 问题分析



#### PHP 代码实现

```php
/**
 * @param Integer $x
 * @return Integer
 */
function reverse($x)
{
    if ($x === 0) return 0;


    $numArr = [];
    $max32Num = 2147483648;
    $isPositive = true;

    if ($x < 0) {
        $isPositive = false;
        $x = -$x;
    }
    while ($x > 0) {
        $numArr[] = $x % 10;
        $x = intval($x / 10);
    }

    $numLength = count($numArr);
    $numPow = pow(10, $numLength - 1);
    $result = 0;
    for ($i = 0; $i < $numLength; $i++) {
        if ($numArr[$i] === 0) {
            $numPow = intval($numPow / 10);
            continue;
        }
        $result += $numArr[$i] * $numPow;
        $numPow = intval($numPow / 10);
    }

    if ($isPositive && $result > $max32Num - 1) return 0;
    if (!$isPositive && $result > $max32Num) return 0;
    if (!$isPositive) $result = -$result;
    return $result;
}
```

测试代码：

```php
$testList = [120];
$resultList = [];

$timeStart = intval(microtime(true) * 1000);
foreach ($testList as $item) {
    $resultList[] = reverse($item);
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

执行用时：12 ms, 在所有 PHP 提交中击败了46.42% 的用户

内存消耗：14.7 MB, 在所有 PHP 提交中击败了100.00% 的用户

## 参考来源

#### [7. 整数反转](https://leetcode-cn.com/problems/reverse-integer/)