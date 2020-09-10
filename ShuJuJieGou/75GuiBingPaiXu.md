```json
{
  "updated_at": "2020-07-11",
  "updated_by": "KelipuTe",
  "tags": "数据结构,Data Structure,算法,Algorithm,排序,Sort"
}
```

---

## 归并排序

归并排序（Merge Sort）是建立在归并操作上的一种有效的排序算法。

该算法是采用分治法（Divide and Conquer）的一个非常典型的应用。将已有序的子序列合并，得到完全有序的序列。即先使每个子序列有序，再使子序列段间有序。若合并步骤是将两个有序表合并成一个有序表，称为2-路归并。 

归并排序是一种稳定的排序方法。和选择排序一样，归并排序的性能不受输入数据的影响，但表现比选择排序好的多，因为始终都是 $O(n \log_2(n))$ 的时间复杂度。代价是需要额外的内存空间，空间复杂度 $T(n)$。

### 排序步骤

算法步骤如下：

- 把长度为n的输入序列分成两个长度为n/2的子序列；
- 对这两个子序列分别采用归并排序；
- 将两个排序好的子序列合并成一个最终的排序序列。

### 动图演示

<img src="E:\GongZuoQu\KTZhiShiKu\Image\ShuJuJieGou\GuiBingPaiXu_img01.gif" style="zoom:67%;" />

### 归并排序算法

这个算法里交换元素的方法和上图的方法不一样，我推荐直接打印结果出来，然后再理解过程。

```php
/**
 * Class GuiBingPaiXu [归并排序]
 * @package App\ShuJuJieGou
 */
class GuiBingPaiXu extends PaiXuAbstract
{
    protected function doSort()
    {
        $this->afterSortList = $this->beforeSortList;
        $startIndex = 0;
        $endIndex = count($this->afterSortList) - 1;
        $this->guiBingPaiXu($startIndex, $endIndex);
    }

    protected function guiBingPaiXu($startIndex, $endIndex)
    {
        ++$this->sortTimes;
        if ($startIndex < $endIndex) {
            $middleIndex = intval(floor(($startIndex + $endIndex) / 2));
            $this->guiBingPaiXu($startIndex, $middleIndex);
            $this->guiBingPaiXu($middleIndex + 1, $endIndex);
            $this->heBingXuLie($startIndex, $middleIndex, $endIndex);
        }
    }

    protected function heBingXuLie($startIndex, $middleIndex, $endIndex)
    {
        $i = $startIndex;
        $j = $middleIndex + 1;
        $k = $startIndex;
        $tempArray = [];
        while ($i !== $middleIndex + 1 && $j !== $endIndex + 1) {
            // 当两个序列都没有到头，需要比较哪个序列的第一个元素更小
            if ($this->afterSortList[$i] >= $this->afterSortList[$j]) {
                $tempArray[$k++] = $this->afterSortList[$j++];
            } else {
                $tempArray[$k++] = $this->afterSortList[$i++];
            }
        }
        while ($i !== $middleIndex + 1) {
            // 当前面的序列已经到头，直接把后面序列的元素依次赋值
            $tempArray[$k++] = $this->afterSortList[$i++];
        }
        while ($j !== $endIndex + 1) {
            // 当后面的序列已经到头，直接把前面序列的元素依次赋值
            $tempArray[$k++] = $this->afterSortList[$j++];
        }
        for ($i = $startIndex; $i <= $endIndex; $i++) {
            // 把排序好的序列替换到原序列的位置
            $this->afterSortList[$i] = $tempArray[$i];
        }
        $this->sortSteps[] = json_encode($this->afterSortList);
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
$xuanZePaiXu = new GuiBingPaiXu($beforeSortList);
echo json_encode($xuanZePaiXu->getSortResult());
```