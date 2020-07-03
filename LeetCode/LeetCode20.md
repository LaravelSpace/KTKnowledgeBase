```json
{
  "updated_by": "KelipuTe",
  "updated_at": "2020-07-03",
  "tags": "LeetCode,算法"
}
```

---

## 20. 有效的括号

#### 问题描述

给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串，判断字符串是否有效。

有效字符串需满足：

    左括号必须用相同类型的右括号闭合。
    左括号必须以正确的顺序闭合。

注意空字符串可被认为是有效字符串。

示例 1:

```
输入: "()"
输出: true
```

示例 2:

```
输入: "()[]{}"
输出: true
```

示例 3:

```
输入: "(]"
输出: false
```

示例 4:

```
输入: "([)]"
输出: false
```

示例 5:

```
输入: "{[]}"
输出: true
```

#### 问题分析

使用栈的思想，如果遇到 '{','[','('，就入栈，遇到 ')',']','}'，就从栈里取出最上层元素，进行匹配。

如果匹配不成对，则说明存在交叉的括号。如果扫描结束，栈里还有没有匹配的括号，则说明存在落单的括号。

#### PHP 代码实现

```php
/**
 * @param String $s
 * @return Boolean
 */
function isValid($s)
{
    $wait = [];
    $waitLen = 0;
    for ($i = 0; $i < strlen($s); $i++) {
        if ($s[$i] === '(' || $s[$i] === '[' || $s[$i] === '{') {
            $wait[] = $s[$i];
            $waitLen++;
            continue;
        }
        if ($s[$i] !== ')' && $s[$i] !== ']' && $s[$i] !== '}') {
            continue;
        } else if ($s[$i] === ')' && $wait[$waitLen - 1] === '(') {
            array_pop($wait);
            $waitLen--;
        } else if ($s[$i] === ']' && $wait[$waitLen - 1] === '[') {
            array_pop($wait);
            $waitLen--;
        } else if ($s[$i] === '}' && $wait[$waitLen - 1] === '{') {
            array_pop($wait);
            $waitLen--;
        } else {
            return false;
        }
    }
    if ($waitLen) return false;
    return true;
}
```

测试代码：

```php
$testList = ['()', '()[]{}', '(]', '([)]', '{[]}', 'a', 'a(a)a'];
$resultList = [];

$timeStart = intval(microtime(true) * 1000);
foreach ($testList as $item) {
    $resultList[] = isValid($item);
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

执行用时：4 ms, 在所有 PHP 提交中击败了95.20% 的用户

内存消耗：15.2 MB, 在所有 PHP 提交中击败了100.00% 的用户

## 参考来源

#### [20. 有效的括号](https://leetcode-cn.com/problems/valid-parentheses/)