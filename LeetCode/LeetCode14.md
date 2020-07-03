```json
{
  "updated_by": "KelipuTe",
  "updated_at": "2020-07-03",
  "tags": "LeetCode,算法"
}
```

---

## 14. 最长公共前缀

#### 问题描述

编写一个函数来查找字符串数组中的最长公共前缀。

如果不存在公共前缀，返回空字符串 ""。

示例 1:

```
输入: ["flower","flow","flight"]
输出: "fl"
```

示例 2:

```
输入: ["dog","racecar","car"]
输出: ""
解释: 输入不存在公共前缀。
```

说明:

所有输入只包含小写字母 a-z 。

#### 问题分析

逐位比较每个字符串的同位的字符是不是一样，如果不一样就结束比较。

#### PHP 代码实现

```php
/**
 * @param String[] $strs
 * @return String
 */
function longestCommonPrefix($strs)
{
    if (empty($strs)) return '';
    if (!isset($strs[1])) return $strs[0];

    $resultStr = '';
    $i = 0;
    $doContinue = true;
    while ($doContinue) {
        if (!isset($strs[0][$i])) break;
        $char = $strs[0][$i];
        foreach ($strs as $item) {
            if (!isset($item[$i]) || $char !== $item[$i]) {
                $doContinue = false;
                break;
            }
        }
        if ($doContinue) $resultStr .= $strs[0][$i];
        $i++;
    }

    return $resultStr;
}
```

测试代码：

```php
$testList = [
    [],
    ['floasdafwer'],
    ['floasdafwer', 'flaadfow', 'fladfadsfight', 'flaadaszfow', 'flaadaseddfow'],
    ['apwsedple', 'apadsfasdsd', 'apfvasedfarfgvarfinh'],
    ['aasdfcsv', 'drfgewsedfr', 'awdadaagfaedf', 'awdadaagfaedf'],
    ['aaawdawdawd', 'aaawdawdawd', 'aaawdawdawd', 'aaawdawdawd', 'aaawdawdawd'],
    ['aiasedfrvbhiuhergbfiaerbgfhi', 'vaiedfkcbgkrdeufgbuia', 'aqiwdgbikehd'],
];
$resultList = [];

$timeStart = intval(microtime(true) * 1000);
foreach ($testList as $item) {
    $resultList[] = longestCommonPrefix($item);
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

执行用时：12 ms, 在所有 PHP 提交中击败了48.05 的用户

内存消耗：14.8 MB, 在所有 PHP 提交中击败了100.00% 的用户

## 参考来源

#### [14. 最长公共前缀](https://leetcode-cn.com/problems/longest-common-prefix/)