```json
{
  "updated_by": "KelipuTe",
  "updated_at": "2020-07-03",
  "tags": "LeetCode,算法"
}
```

---

## 125. 验证回文串

#### 问题描述

给定一个字符串，验证它是否是回文串，只考虑字母和数字字符，可以忽略字母的大小写。

说明：本题中，我们将空字符串定义为有效的回文串。

示例 1:

```
输入: "A man, a plan, a canal: Panama"
输出: true
```

示例 2:

```
输入: "race a car"
输出: false
```

#### 问题分析

题目的要求是要去掉空格和特殊符号的，同时还要忽略大小写。

#### PHP 代码实现

```php
/**
 * @param String $s
 * @return Boolean
 */
function isPalindrome($s)
{
    if (strlen($s) === 0) return true;
    $s = preg_replace('/\W/', '', $s);
    $s = strtolower($s);
    $length = strlen($s);
    for ($i = 0; $i < $length / 2; $i++) {
        if ($s[$i] !== $s[$length - 1 - $i]) return false;
    }
    
    return true;
}
```

测试代码：

```php
$testList = [
    'A man, a plan, a canal: Panama',
    'race a car',
    'abccba',
    'abcdcba',
    'ab1cdc1ba',
    'ab1cdc2ba',
];
$resultList = [];

$timeStart = intval(microtime(true) * 1000);
foreach ($testList as $item) {
    $resultList[] = isPalindrome($item);
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

执行用时：16 ms, 在所有 PHP 提交中击败了60.59% 的用户

内存消耗：15.3 MB, 在所有 PHP 提交中击败了100.00% 的用户

## 参考来源

#### [125. 验证回文串](https://leetcode-cn.com/problems/valid-palindrome/)