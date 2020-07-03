```json
{
  "updated_by": "KelipuTe",
  "updated_at": "2020-07-03",
  "tags": "LeetCode,算法"
}
```

---

## 38. 外观数列

#### 问题描述

给定一个正整数 n（1 ≤ n ≤ 30），输出外观数列的第 n 项。

注意：整数序列中的每一项将表示为一个字符串。

「外观数列」是一个整数序列，从数字 1 开始，序列中的每一项都是对前一项的描述。前五项如下：

```
1.     1
2.     11
3.     21
4.     1211
5.     111221
```

第一项是数字 1

描述前一项，这个数是 1 即 “一个 1 ”，记作 11

描述前一项，这个数是 11 即 “两个 1 ” ，记作 21

描述前一项，这个数是 21 即 “一个 2 一个 1 ” ，记作 1211

描述前一项，这个数是 1211 即 “一个 1 一个 2 两个 1 ” ，记作 111221

示例 1:

```
输入: 1
输出: "1"
解释：这是一个基本样例。
```

示例 2:

```
输入: 4
输出: "1211"
解释：当 n = 3 时，序列是 "21"，其中我们有 "2" 和 "1" 两组，"2" 可以读作 "12"，也就是出现频次 = 1 而 值 = 2；类似 "1" 可以读作 "11"。所以答案是 "12" 和 "11" 组合在一起，也就是 "1211"。
```

#### 问题分析

题目的意思就是，计算一个数字字符转里面连续的数字有几个，然后输出成一个新的数字字符串。

#### PHP 代码实现

```php
/**
 * @param Integer $n
 * @return String
 */
function countAndSay($n)
{
    $numStrList = [];
    $numStrList[0] = '1';
    $numStrList[1] = '11';
    for ($i = 2; $i < $n; $i++) {
        $oleNumStr = $numStrList[$i - 1];
        $newNumStr = '';
        $num = $oleNumStr[0];
        $count = 1;
        for ($j = 1; $j < strlen($oleNumStr); $j++) {
            if ($num === $oleNumStr[$j]) {
                $count++;
            } else {
                $newNumStr .= $count . $num;
                $num = $oleNumStr[$j];
                $count = 1;
            }
        }
        $newNumStr .= $count . $num;
        $numStrList[$i] = $newNumStr;
    }
    
    return $numStrList[$n - 1];
}
```

测试代码：

```php
$testList = [5];
$resultList = [];

$timeStart = intval(microtime(true) * 1000);
foreach ($testList as $item) {
    $resultList[] = countAndSay($item);
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

执行用时：8 ms, 在所有 PHP 提交中击败了80.91% 的用户

内存消耗：15 MB, 在所有 PHP 提交中击败了100.00% 的用户

## 参考来源

#### [38. 外观数列](https://leetcode-cn.com/problems/count-and-say/)