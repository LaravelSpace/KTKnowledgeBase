```json
{
  "updated_by": "KelipuTe",
  "updated_at": "2020-07-03",
  "tags": "LeetCode,算法"
}
```

---

## 13. 罗马数字转整数

#### 问题描述

罗马数字包含以下七种字符: `I`， `V`， `X`， `L`，`C`，`D` 和 `M`。

```
字符          数值
I             1
V             5
X             10
L             50
C             100
D             500
M             1000
```

例如， 罗马数字 2 写做 II ，即为两个并列的 1。12 写做 XII ，即为 X + II 。 27 写做  XXVII, 即为 XX + V + II 。

通常情况下，罗马数字中小的数字在大的数字的右边。但也存在特例，例如 4 不写做 IIII，而是 IV。数字 1 在数字 5 的左边，所表示的数等于大数 5 减小数 1 得到的数值 4 。同样地，数字 9 表示为 IX。这个特殊的规则只适用于以下六种情况：

    I 可以放在 V (5) 和 X (10) 的左边，来表示 4 和 9。
    X 可以放在 L (50) 和 C (100) 的左边，来表示 40 和 90。 
    C 可以放在 D (500) 和 M (1000) 的左边，来表示 400 和 900。

给定一个罗马数字，将其转换成整数。输入确保在 1 到 3999 的范围内。

示例 1:

```
输入: "III"
输出: 3
```

示例 2:

```
输入: "IV"
输出: 4
```

示例 3:

```
输入: "IX"
输出: 9
```

示例 4:

```
输入: "LVIII"
输出: 58
解释: L = 50, V= 5, III = 3.
```

示例 5:

```
输入: "MCMXCIV"
输出: 1994
解释: M = 1000, CM = 900, XC = 90, IV = 4.
```

#### 问题分析

通常情况下直接累加所有的数字即可。

XXV=10+10+5；XXVI=10+10+5+1；

对于特例，则可以通过判断左右两个字符的大小来识别。当左边的字符比右边的字符小时，减去左边的字符即可。

XXIII=10+10+1+1+1；XXIV=10+10-1+5；

#### PHP 代码实现

```php
/**
 * @param String $s
 * @return Integer
 */
function romanToInt($s)
{
    $romanMap = ['I' => 1, 'V' => 5, 'X' => 10, 'L' => 50, 'C' => 100, 'D' => 500, 'M' => 1000];
    $sArr = str_split($s);
    $n = 0;
    for ($i = 0; $i < count($sArr) - 1; $i++) {
        if ($romanMap[$sArr[$i]] >= $romanMap[$sArr[$i + 1]]) {
            $n += $romanMap[$sArr[$i]];
        } else {
            $n -= $romanMap[$sArr[$i]];
        }
    }
    $n += $romanMap[$sArr[count($sArr) - 1]];

    return $n;
}
```

测试代码：

```php
$testList = ['I', 'IV', 'V', 'VI', 'XL', 'XLI', 'XXIV', 'XXV', 'XXVI'];
$resultList = [];

$timeStart = intval(microtime(true) * 1000);

foreach ($testList as $item) {
    $resultList[] = romanToInt($item);
}
$timeStop = intval(microtime(true) * 1000);

$echo = [
    'timeStart' => $timeStart,
    'timeStop'  => $timeStop,
    'timePass'  => $timeStop - $timeStart,
    'result'    => $resultList
];
echo json_encode($echo);
```

#### 测试结果

执行结果：通过

执行用时：16 ms, 在所有 PHP 提交中击败了89.33% 的用户

内存消耗：14.5 MB, 在所有 PHP 提交中击败了100.00% 的用户

## 参考来源

#### [13. 罗马数字转整数](https://leetcode-cn.com/problems/roman-to-integer/)