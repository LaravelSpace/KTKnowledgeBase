```json
{
  "updated_by": "KelipuTe",
  "updated_at": "2020-07-01",
  "tags": "LeetCode,算法,动态规划"
}
```

## 718.最长重复子数组

#### 问题描述

给两个整数数组 A 和 B ，返回两个数组中公共的、长度最长的子数组的长度。

示例：

```
输入：
A: [1,2,3,2,1]
B: [3,2,1,4,7]
输出：3
解释：
长度最长的公共子数组是 [3, 2, 1] 。
```

提示：

```
1 <= len(A), len(B) <= 1000
0 <= A[i], B[i] < 100
```

#### 问题分析

暴力穷举的方式可以解决，依次分别从 A[i]（0<=i<len(A))）和 B[j]（0<=j<len(B)） 开始，比较二者剩余部分的最长公共前缀序列。时间复杂度 $O(n^3)$。

这题还可以使用动态规划的方法，反过来思考，从末尾开始。比较 A[i] 和 B[j] ，记问题的结果为 `dp[i][j]`。无论 A[i] 和 B[j] 相不相同，问题的结果 `dp[i][j]` 都依赖于，数组 A A[i] 前面的部分和数组 B B[j] 的前面的部分关于此问题的结果即 `dp[i-1][j-1]`。依次类推我们就可以找动态规划的状态转移方程。

![ZuiChangChongFuZiShuZu_01](./ZuiChangChongFuZiShuZu_01.png)

下面给出的是状态转移表（第一列表示 A 数组，第一行表示 B 数组）。

|      | 3    | 2    | 1    | 4    | 7    |
| :--- | ---- | ---- | ---- | ---- | ---- |
| 1    | 0    | 0    | 1    | 0    | 0    |
| 2    | 0    | 1    | 0    | 0    | 0    |
| 3    | 1    | 0    | 0    | 0    | 0    |
| 2    | 0    | 2    | 0    | 0    | 0    |
| 1    | 0    | 0    | 3    | 0    | 0    |

比如，{A2,B2} 这个位置的结果，就依赖于 {A3,B1} 这个位置的结果，表示 [3,2] 和 [1,2] 的最长公共子数组的结果依赖 [3] 和 [1] 的最长公共子数组的结果。值得注意的是 {A2,B1} 这个位置的结果，可以理解成依赖于 {A1,B0} 也就是 [3] 和 [] 的结果，[3] 和 [] 的结果当然是 0。

#### PHP 代码实现

```php
/**
 * @param Integer[] $A
 * @param Integer[] $B
 * @return Integer
 */
function findLength($A, $B)
{
    $lenA = count($A);
    if ($lenA === 0) return 0;
    $lenB = count($B);
    if ($lenB === 0) return 0;

    $dp = [];
    $maxLen = 0;
    // 初始化第一行和第一列的结果
    for ($j = 0; $j < $lenB; ++$j) {
        $dp[0][$j] = $B[$j] === $A[0] ? 1 : 0;
        $maxLen = $dp[0][$j] > $maxLen ? $dp[0][$j] : $maxLen;
    }
    for ($i = 0; $i < $lenA; ++$i) {
        $dp[$i][0] = $A[$i] === $B[0] ? 1 : 0;
        $maxLen = $dp[$i][0] > $maxLen ? $dp[$i][0] : $maxLen;
    }
    // 通过状态转移方程计算结果
    for ($i = 1; $i < $lenA; ++$i) {
        for ($j = 1; $j < $lenB; ++$j) {
            $dp[$i][$j] = $A[$i] === $B[$j] ? $dp[$i - 1][$j - 1] + 1 : 0;
            $maxLen = $dp[$i][$j] > $maxLen ? $dp[$i][$j] : $maxLen;
        }
    }

    return $maxLen;
}
```

测试代码：

```php
$testList = [
    ['A' => [], 'B' => []],
    ['A' => [], 'B' => [1, 2]],
    ['A' => [1, 2, 1, 2], 'B' => []],
    ['A' => [1, 2, 1, 2], 'B' => [1, 2]],
    ['A' => [1, 2, 3, 2, 1], 'B' => [3, 2, 1, 4, 7]],
    ['A' => [1, 2, 3, 3, 2, 1], 'B' => [3, 2, 7, 1, 4, 7]],
    ['A' => [1, 2, 3, 2, 1], 'B' => [3, 2, 1, 4, 7, 1, 2, 3, 2, 1]],
];
$resultList = [];

$timeStart = intval(microtime(true) * 1000);
foreach ($testList as $item) {
    $resultList[] = findLength($item['A'], $item['B']);
}
$timeStop = intval(microtime(true) * 1000);

$echo = [
    'timeStart' => $timeStart,
    'timeStop' => $timeStop,
    'timePass' => ($timeStop - $timeStart),
    'resultList' => $resultList
];
echo json_encode($echo);
```

#### 测试结果

执行结果：通过

执行用时：1340 ms, 在所有 PHP 提交中击败了40.00% 的用户

内存消耗：47 MB, 在所有 PHP 提交中击败了100.00% 的用户

## 参考来源

#### [718. 最长重复子数组](https://leetcode-cn.com/problems/maximum-length-of-repeated-subarray/)

[【手绘图解】DP 和 滑动窗口](https://leetcode-cn.com/problems/maximum-length-of-repeated-subarray/solution/zhe-yao-jie-shi-ken-ding-jiu-dong-liao-by-hyj8/)