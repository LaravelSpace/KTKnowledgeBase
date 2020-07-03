```json
{
  "updated_by": "KelipuTe",
  "updated_at": "2020-07-03",
  "tags": "LeetCode,算法"
}
```

---

## 58. 最后一个单词的长度

#### 问题描述

给定一个仅包含大小写字母和空格 ' ' 的字符串 s，返回其最后一个单词的长度。如果字符串从左向右滚动显示，那么最后一个单词就是最后出现的单词。

如果不存在最后一个单词，请返回 0 。

说明：一个单词是指仅由字母组成、不包含任何空格字符的 最大子字符串。

示例:

```
输入: "Hello World"
输出: 5
```

#### 问题分析



#### PHP 代码实现

```php
/**
 * @param String $s
 * @return Integer
 */
function lengthOfLastWord($s)
{
    $length = strlen($s);
    $start = false;
    $str = '';
    for ($i = $length - 1; $i > -1; $i--) {
        if (!$start && $s[$i] === ' ') {
            continue;
        } else {
            $start = true;
        }
        if ($start && $s[$i] !== ' ') {
            $str = $s[$i] . $str;
        } else {
            break;
        }
    }

    return strlen($str);
}
```

测试代码：

```php
$testList = ['', ' ', ' aa bb ', 'array list', 'hello world'];
$resultList = [];

$timeStart = intval(microtime(true) * 1000);
foreach ($testList as $item) {
    $resultList[] = lengthOfLastWord($item);
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

执行用时：8 ms, 在所有 PHP 提交中击败了64.17 的用户

内存消耗：15 MB, 在所有 PHP 提交中击败了100.00% 的用户

## 参考来源

#### [58. 最后一个单词的长度](https://leetcode-cn.com/problems/length-of-last-word/)