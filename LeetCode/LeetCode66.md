```json
{
  "updated_by": "KelipuTe",
  "updated_at": "2020-07-03",
  "tags": "LeetCode,算法"
}
```

---

## 66. 加一

#### 问题描述

给定一个由整数组成的非空数组所表示的非负整数，在该数的基础上加一。

最高位数字存放在数组的首位， 数组中每个元素只存储单个数字。

你可以假设除了整数 0 之外，这个整数不会以零开头。

示例 1:

```
输入: [1,2,3]
输出: [1,2,4]
解释: 输入数组表示数字 123。
```

示例 2:

```
输入: [4,3,2,1]
输出: [4,3,2,2]
解释: 输入数组表示数字 4321。
```

#### 问题分析

主要就是要注意进位的时候。

#### PHP 代码实现

```php
/**
 * @param Integer[] $digits
 * @return Integer[]
 */
function plusOne($digits)
{
    $length = count($digits);
    if ($digits[$length - 1] === 9) {
        if (!isset($digits[$length - 2])) return [1, 0];
        $digits[$length - 1] = 0;
        $digits[$length - 2]++;
        for ($i = $length - 2; $i > 0; $i--) {
            if ($digits[$i] === 10) {
                $digits[$i] = 0;
                $digits[$i - 1]++;
            }
        }
        if ($digits[0] === 10) {
            $digits[0] = 0;
            array_unshift($digits, 1);
        }
    } else {
        $digits[$length - 1]++;
    }

    return $digits;
}
```

测试代码：

```php
$testList = [
    [0], [9],
    [1, 9], [9, 9],
    [1, 1, 9], [1, 9, 9],
    [9, 9, 9, 9],
];
$resultList = [];

$timeStart = intval(microtime(true) * 1000);
foreach ($testList as $item) {
    $resultList[] = plusOne($item);
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

内存消耗：14.9 MB, 在所有 PHP 提交中击败了100.00% 的用户

## 参考来源

#### [66. 加一](https://leetcode-cn.com/problems/plus-one/)