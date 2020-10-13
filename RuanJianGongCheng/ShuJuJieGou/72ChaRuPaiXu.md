```json
{
  "updated_at": "2020-07-11",
  "updated_by": "KelipuTe",
  "tags": "数据结构,Data Structure,算法,Algorithm,排序,Sort"
}
```

---

## 插入排序

插入排序（Insertion Sort）是一种简单直观的排序算法。

它的工作原理：通过构建有序序列，对于未排序数据，在已排序序列中从后向前扫描，找到相应位置并插入。

时间复杂度 $O(n^2)$，空间复杂度 $T(1)$。

### 排序步骤

一般来说，插入排序都采用in-place在数组上实现。具体算法描述如下：

- 从第一个元素开始，该元素可以认为已经被排序；
- 取出下一个元素，在已经排序的元素序列中从后向前扫描；
- 如果该元素（已排序）大于新元素，将该元素移到下一位置；
- 重复步骤3，直到找到已排序的元素小于或者等于新元素的位置；
- 将新元素插入到该位置后；
- 重复步骤2~5。

### 动图演示

<img src="E:\GongZuoQu\KTZhiShiKu\Image\ShuJuJieGou\ChaRuPaiXu_img01.gif" style="zoom:67%;" />

### 选择排序算法

```php
/**
 * Class ChaRuPaiXu [插入排序]
 * @package App\ShuJuJieGou
 */
class ChaRuPaiXu extends PaiXuAbstract
{
    protected function doSort()
    {
        $afterSortList = $this->beforeSortList;
        $length = count($afterSortList);
        for ($i = 1; $i < $length; ++$i) {
            $j = $i;
            $tempNum = $afterSortList[$j];
            while ($j > 0 && $tempNum < $afterSortList[$j - 1]) {
                ++$this->sortTimes;
                $afterSortList[$j] = $afterSortList[$j - 1];
                --$j;
            }
            $afterSortList[$j] = $tempNum;
            $this->sortSteps[] = json_encode($afterSortList);
        }
        $this->afterSortList = $afterSortList;
    }
}
```

测试代码：

```php
$beforeSortList = [];
$length = 10;
for ($i = 0; $i < $length; $i++) {
    $beforeSortList[] = rand(1, 50);
}
$xuanZePaiXu = new ChaRuPaiXu($beforeSortList);
echo json_encode($xuanZePaiXu->getSortResult());
```