```json
{
  "updated_at": "2020-07-11",
  "updated_by": "KelipuTe",
  "tags": "数据结构,Data Structure,算法,Algorithm,排序,Sort"
}
```

---

## 冒泡排序

冒泡排序（Bubble Sort）是一种简单的排序算法。

它的工作原理：重复地走访过要排序的数列，一次比较两个元素，如果它们的顺序错误就把它们交换过来。走访数列的工作是重复地进行直到没有再需要交换，也就是说该数列已经排序完成。这个算法的名字由来是因为越小的元素会经由交换慢慢“浮”到数列的顶端。

时间复杂度 $O(n^2)$，空间复杂度 $T(1)$。

### 排序步骤

算法步骤如下：

- 比较相邻的元素。如果第一个比第二个大，就交换它们两个；
- 对每一对相邻元素作同样的工作，从开始第一对到结尾的最后一对，这样在最后的元素应该会是最大的数；
- 针对所有的元素重复以上的步骤，除了最后一个；
- 重复步骤1~3，直到排序完成。

### 动图演示

<img src="E:\GongZuoQu\KTZhiShiKu\Image\ShuJuJieGou\MaoPaoPaiXu_img01.gif" style="zoom:67%;" />

### 冒泡排序算法

```php
/**
 * Class MaoPaoPaiXu [冒泡排序]
 * @package App\ShuJuJieGou
 */
class MaoPaoPaiXu extends PaiXuAbstract
{
    protected function doSort()
    {
        $afterSortList = $this->beforeSortList;
        $length = count($afterSortList);
        // 记录最后一个被交换的元素的位置
        $lastExchangeIndex = $length - 1;
        for ($i = 0; $i < $length - 1; $i++) {
            // 记录数组中是否有元素被交换
            $isSorted = false;
            // 下一轮内层循环，只比较到最后一个被交换的元素的位置
            $lastSortIndex = $lastExchangeIndex;
            for ($j = 0; $j < $lastSortIndex; $j++) {
                ++$this->sortTimes;
                if ($afterSortList[$j] > $afterSortList[$j + 1]) {
                    $tempNum = $afterSortList[$j + 1];
                    $afterSortList[$j + 1] = $afterSortList[$j];
                    $afterSortList[$j] = $tempNum;
                    $this->sortSteps[] = json_encode($afterSortList);

                    $isSorted = true;
                    $lastExchangeIndex = $j;
                }
            }
            if (!$isSorted) {
                // 如果数组中没有元素被交换，则数组已经有序
                break;
            }
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
$xuanZePaiXu = new MaoPaoPaiXu($beforeSortList);
echo json_encode($xuanZePaiXu->getSortResult());
```