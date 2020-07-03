```json
{
  "updated_by": "KelipuTe",
  "updated_at": "2020-07-03",
  "tags": "LeetCode,算法"
}
```

---

## 67. 二进制求和

#### 问题描述

给你两个二进制字符串，返回它们的和（用二进制表示）。

输入为 **非空** 字符串且只包含数字 `1` 和 `0`。

示例 1:

```
输入: a = "11", b = "1"
输出: "100"
```

示例 2:

```
输入: a = "1010", b = "1011"
输出: "10101"
```

提示：

    每个字符串仅由字符 '0' 或 '1' 组成。
    1 <= a.length, b.length <= 10^4
    字符串如果不是 "0" ，就都不含前导零。

#### 问题分析

把字符串换转换成数组，然后从最后一位开始算。只有求和有四种结果，分别是 0,1,2,3。0 和 1 是不需要考虑进位的，2 对应 进 1 余 0 ，3 对应 进 1 余 1。进位的结果直接加在前面一个位置的结果上，重复上述求和操作。

#### PHP 代码实现

```php
/**
 * @param String $a
 * @param String $b
 * @return String
 */
function addBinary($a, $b)
{
    $a = str_split($a);
    $b = str_split($b);
    $lengthA = count($a);
    $lengthB = count($b);
    if ($lengthA < $lengthB) {
        $long = $b;
        $lengthL = $lengthB;
        $short = $a;
        $lengthS = $lengthA;
    } else {
        $long = $a;
        $lengthL = $lengthA;
        $short = $b;
        $lengthS = $lengthB;
    }
    for ($i = $lengthL - 1, $j = $lengthS - 1; $i > 0; $i--, $j--) {
        if ($j > -1) {
            $long[$i] = intval($long[$i]) + intval($short[$j]);
        }
        if (intval($long[$i]) === 2) {
            $long[$i] = 0;
            $long[$i - 1] = intval($long[$i - 1]) + 1;
        } else if (intval($long[$i]) === 3) {
            $long[$i] = 1;
            $long[$i - 1] = intval($long[$i - 1]) + 1;
        }
    }
    if ($lengthL === $lengthS) {
        $long[0] = intval($long[0]) + intval($short[0]);
    }
    if ($long[0] === 2) {
        $long[0] = 0;
        array_unshift($long, 1);
    }
    if ($long[0] === 3) {
        $long[0] = 1;
        array_unshift($long, 1);
    }
    
    return implode($long);
}
```

测试代码：

```php
$testList = [
    ['a' => '11', 'b' => '1'],
    ['a' => '1', 'b' => '10'],
    ['a' => '10', 'b' => '110'],
    ['a' => '1011', 'b' => '110'],
    ['a' => '1010', 'b' => '1011']
];
$resultList = [];

$timeStart = intval(microtime(true) * 1000);
foreach ($testList as $item) {
    $resultList[] = addBinary($item['a'], $item['b']);
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

执行用时：8 ms, 在所有 PHP 提交中击败了85.44% 的用户

内存消耗：15.1 MB, 在所有 PHP 提交中击败了100.00% 的用户

## 参考来源

#### [67. 二进制求和](https://leetcode-cn.com/problems/add-binary/)