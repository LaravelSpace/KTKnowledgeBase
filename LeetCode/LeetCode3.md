```json
{
  "updated_by": "KelipuTe",
  "updated_at": "2020-07-05",
  "tags": "LeetCode,算法"
}
```

---

## 3. 无重复字符的最长子串

#### 问题描述

给定一个字符串，请你找出其中不含有重复字符的 最长子串 的长度。

示例 1:

```
输入: "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
```

示例 2:

```
输入: "bbbbb"
输出: 1
解释: 因为无重复字符的最长子串是 "b"，所以其长度为 1。
```

示例 3:

```
输入: "pwwkew"
输出: 3
解释: 因为无重复字符的最长子串是 "wke"，所以其长度为 3。
     请注意，你的答案必须是 子串 的长度，"pwke" 是一个子序列，不是子串。
```

#### 问题分析

使用滑动窗口。

我们不妨以示例一中的字符串 abcabcbb 为例，找出 从每一个字符开始的，不包含重复字符的最长子串，那么其中最长的那个字符串即为答案。对于示例一中的字符串，我们列举出这些结果，其中括号中表示选中的字符以及最长的字符串：

- 以 (a)bcabcbb 开始的最长字符串为 (abc)abcbb；
- 以 a(b)cabcbb 开始的最长字符串为 a(bca)bcbb；
- 以 ab(c)abcbb 开始的最长字符串为 ab(cab)cbb；
- 以 abc(a)bcbb 开始的最长字符串为 abc(abc)bb；
- 以 abca(b)cbb 开始的最长字符串为 abca(bc)bb；
- 以 abcab(c)bb 开始的最长字符串为 abcab(cb)b；
- 以 abcabc(b)b 开始的最长字符串为 abcabc(b)b；
- 以 abcabcb(b) 开始的最长字符串为 abcabcb(b)；

如果我们依次递增地枚举子串的起始位置，那么子串的结束位置也是递增的。假设我们选择字符串中的第 k 个字符作为起始位置，并且得到了不包含重复字符的最长子串的结束位置为 r。那么当我们选择第 k+1 个字符作为起始位置时，首先从 k+1 到 r 的字符显然是不重复的，并且由于少了原本的第 k 个字符，我们可以尝试继续增大 r，直到右侧出现了重复字符为止。这样，我们就可以使用滑动窗口来解决这个问题了。另外还需要一个唯一集合判断字符串是否重复。

以字符串 pwwkew 举例。给出判断逻辑。

| 起始条件            | 判断过程和结果        |
| ------------------- | --------------------- |
| k=0,r=0,set=[]      | set=[p],++r;          |
| k=0,r=1,set=[p]     | set=[p,w],++r;        |
| k=0,r=2,set=[p,w]   | s[2]=w,++k,set=[w];   |
| k=1,r=2,set=[w]     | s[2]=w,++k,set=[];    |
| k=2,r=2,set=[]      | set=[w];++r           |
| k=2,r=3,set=[w]     | set=[w,k];++r         |
| k=2,r=4,set=[w,k]   | set=[w,k,e];++r       |
| k=2,r=5,set=[w,k,e] | s[5]=w,++k,set=[k,e]; |
| k=3,r=5,set=[k,e]   | set=[k,e,w];++r       |

#### PHP 代码实现

```php
/**
 * @param String $s
 * @return Integer
 */
function lengthOfLongestSubstring($s)
{
    $len = strlen($s);
    if ($len === 0) return 0;
    if ($len === 1) return 1;

    // 右指针 $p，初始化为 -1 表示还没有移动
    $p = -1;
    $strLen = 0;
    // set 集合用于判断字符串是否重复
    $strSet = [];
    // 左指针 $i
    for ($i = 0; $i < $len; ++$i) {
        // 当左指针右移的时候要从 set 里移除前一个位置的字符
        if ($i !== 0) unset($strSet[$s[$i - 1]]);
        while ($p + 1 < $len && !isset($strSet[$s[$p + 1]])) {
            // 如果没有重复，指针就继续右移
            $strSet[$s[$p + 1]] = $s[$p + 1];
            ++$p;
        }
        $strLen = max($strLen, $p - $i + 1);
    }

    return $strLen;
}
```

测试代码：

```php
$testList = ['abcabcbb', 'bbbbb', 'pwwkew'];
$resultList = [];

$timeStart = intval(microtime(true) * 1000);
foreach ($testList as $item) {
    $resultList[] = lengthOfLongestSubstring($item);
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

执行用时：24 ms, 在所有 PHP 提交中击败了55.00% 的用户

内存消耗：14.5 MB, 在所有 PHP 提交中击败了100.00% 的用户

## 参考来源

#### [LeetCode](https://leetcode-cn.com/)