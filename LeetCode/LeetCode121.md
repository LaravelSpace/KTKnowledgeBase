```json
{
  "updated_by": "KelipuTe",
  "updated_at": "2020-07-03",
  "tags": "LeetCode,算法"
}
```

---

## 121. 买卖股票的最佳时机

#### 问题描述

给定一个数组，它的第 i 个元素是一支给定股票第 i 天的价格。

如果你最多只允许完成一笔交易（即买入和卖出一支股票一次），设计一个算法来计算你所能获取的最大利润。

注意：你不能在买入股票前卖出股票。


示例 1:

```
输入: [7,1,5,3,6,4]
输出: 5
解释: 在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。
     注意利润不能是 7-1 = 6, 因为卖出价格需要大于买入价格；同时，你不能在买入前卖出股票。
```

示例 2:

```
输入: [7,6,4,3,1]
输出: 0
解释: 在这种情况下, 没有交易完成, 所以最大利润为 0。
```

#### 问题分析

画个价格折线图，更有利于理解。

![LeetCode121_img01](./LeetCode121_img01.png)

记录最大利润和最小买入价格。

因为卖出价格一定时，买入价格越低利润越大。所以每次循环，计算利润并和最大利润比较，如果发现更小买入价格就更新最小买入价格。

位于前面的最小买入价格的结果将会保存在最大利润中，位于后面的最小买入价格只需要和在其之后的价格计算最大利润。

#### PHP 代码实现

```php
/**
 * @param Integer[] $prices
 * @return Integer
 */
function maxProfit($prices)
{
    $maxProfit = 0;
    $minPrice = $prices[0];
    for ($i = 1; $i < count($prices); $i++) {
        $profit = $prices[$i] - $minPrice;
        if ($profit > $maxProfit) $maxProfit = $profit;
        if ($prices[$i] < $minPrice) $minPrice = $prices[$i];
    }

    return $maxProfit;
}
```

测试代码：

```php
$testList = [
    [7, 1, 5, 3, 6, 4],
    [7, 6, 4, 3, 1]
];
$resultList = [];

$timeStart = intval(microtime(true) * 1000);
foreach ($testList as $item) {
    $resultList[] = maxProfit($item);
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

执行用时：16 ms, 在所有 PHP 提交中击败了91.12% 的用户

内存消耗：16.9 MB, 在所有 PHP 提交中击败了100.00% 的用户

## 参考来源

#### [121. 买卖股票的最佳时机](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/)