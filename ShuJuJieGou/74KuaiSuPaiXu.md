```json
{
  "updated_at": "2020-07-11",
  "updated_by": "KelipuTe",
  "tags": "数据结构,Data Structure,算法,Algorithm,排序,Sort"
}
```

---

## 快速排序

快速排序（Quick Sort）使用分治法来把一个序列分为两个序列。

快速排序的平均时间复杂度 $O(n \log_2(n))$，空间复杂度 $T(1)$。在最糟情况下(即待排序序列是有序序列时)，最糟时间复杂度为 $O(n^2)$。

### 排序步骤

算法步骤如下：

- 从数列中挑出一个元素，称为 “基准”（pivot）；
- 重新排序数列，所有元素比基准值小的摆放在基准前面，所有元素比基准值大的摆在基准的后面（相同的数可以到任一边）。在这个分区退出之后，该基准就处于数列的中间位置。这个称为分区（partition）操作；
- 递归地（recursive）把小于基准值元素的子数列和大于基准值元素的子数列排序。

### 动图演示

<img src="E:\GongZuoQu\KTZhiShiKu\Image\ShuJuJieGou\KuaiSuPaiXu_img01.gif" style="zoom:67%;" />

### 快速排序算法

这个算法里交换元素的方法和上图的方法不一样，我推荐直接打印结果出来，然后再理解过程。

```php
/**
 * Class KuaiSuPaiXu [快速排序]
 * @package App\ShuJuJieGou
 */
class KuaiSuPaiXu extends PaiXuAbstract
{
    protected function doSort()
    {
        $this->afterSortList = $this->beforeSortList;
        $startIndex = 0;
        $endIndex = count($this->afterSortList) - 1;
        $this->quickSort($startIndex, $endIndex);
    }

    protected function quickSort($startIndex, $endIndex)
    {
        if ($startIndex >= $endIndex) {
            return;
        }
        $i = $startIndex;
        $j = $endIndex;
        // 设定第一个元素作为基准
        $middleNum = $this->afterSortList[$startIndex];
        // 这里两个循环会来回移动这个基准元素，最后基准元素会把序列分为，前半部分小于基准的，和后半部分大于基准的
        while ($j > $i) {
            ++$this->sortTimes;
            if ($this->afterSortList[$j] < $middleNum) {
                $tempNum = $this->afterSortList[$i];
                $this->afterSortList[$i] = $this->afterSortList[$j];
                $this->afterSortList[$j] = $tempNum;
                $this->sortSteps[] = json_encode($this->afterSortList);
                for ($i += 1; $i < $j; ++$i) {
                    ++$this->sortTimes;
                    if ($this->afterSortList[$i] > $middleNum) {
                        $tempNum = $this->afterSortList[$j];
                        $this->afterSortList[$j] = $this->afterSortList[$i];
                        $this->afterSortList[$i] = $tempNum;
                        $this->sortSteps[] = json_encode($this->afterSortList);
                        break;
                    }
                }
            }
            --$j;
        }
        $this->quickSort($startIndex, $i);
        $this->quickSort($i + 1, $endIndex);
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
$xuanZePaiXu = new KuaiSuPaiXu($beforeSortList);
echo json_encode($xuanZePaiXu->getSortResult());
```