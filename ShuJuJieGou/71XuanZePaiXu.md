```json
{
  "updated_at": "2020-07-17",
  "updated_by": "KelipuTe",
  "tags": "数据结构,Data Structure,算法,Algorithm,排序,Sort"
}
```

---

## 选择排序

选择排序（Selection Sort）是一种简单直观的排序算法。

它的工作原理：首先在未排序序列中找到最小（大）元素，存放到排序序列的起始位置，然后，再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾。以此类推，直到所有元素均排序完毕。

表现最稳定的排序算法之一，无论什么数据进去，时间复杂度都是 $O(n^2)$，空间复杂度都是 $T(1)$。

### 排序步骤

n个记录的直接选择排序可经过n-1趟直接选择排序得到有序结果。具体算法描述如下：

- 初始状态：无序区为R[1..n]，有序区为空；
- 第i趟排序(i=1,2,3…n-1)开始时，当前有序区和无序区分别为R[1..i-1]和R(i..n）。该趟排序从当前无序区中-选出关键字最小的记录 R[k]，将它与无序区的第1个记录R交换，使R[1..i]和R[i+1..n)分别变为记录个数增加1个的新有序区和记录个数减少1个的新无序区；
- n-1趟结束，数组有序化了。

### 动图演示

<img src="E:\Workspace\KTKnowledgeBase\Image\ShuJuJieGou\XuanZePaiXu_img01.gif" style="zoom:67%;" />

### 选择排序算法

```php
/**
 * Class XuanZePaiXu [选择排序]
 */
class XuanZePaiXu extends PaiXuAbstract
{
    protected function doSort()
    {
        $afterSortList = $this->beforeSortList;
        $length = count($afterSortList);
        for ($i = 0; $i < $length; $i++) {
            // 第n次循环找到第n小的那个元素
            $minIndex = $i;
            for ($j = $i + 1; $j < $length; $j++) {
                // 记录排序比较次数
                ++$this->sortTimes;
                if ($afterSortList[$j] < $afterSortList[$minIndex]) {
                    $minIndex = $j;
                }
            }
            // 把第n小的值和序列第n个位置的元素交换
            $tempNum = $afterSortList[$i];
            $afterSortList[$i] = $afterSortList[$minIndex];
            $afterSortList[$minIndex] = $tempNum;
            // 记录排序步骤
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
$xuanZePaiXu = new XuanZePaiXu($beforeSortList);
echo json_encode($xuanZePaiXu->getSortResult());
```
