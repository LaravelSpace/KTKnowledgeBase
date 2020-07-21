```json
{
  "updated_at": "2020-07-18",
  "updated_by": "KelipuTe",
  "tags": "数据结构,Data Structure,图,Graph"
}
```

## 克鲁斯卡尔算法

克鲁斯卡尔算法是用来寻找带权重无向图的最小生成树的。与普里姆算法的区别在于，普里姆算法从顶点着手，克鲁斯卡尔算法从边着手。

过程是这样的：

- 1、首先把无向图中所有的边按照权重从小到大排序。
- 2、找出最小权重的边，判断两个顶点是不是已经在最小生成树上了，如果已经在生成树上了，这个地方的边连起来就会形成环状结构。
- 3、重复上述步骤2，直到所有的顶点都被连进来。

克鲁斯卡尔算法并不需要邻接矩阵或者邻接表。它需要另外一个东西，按权重排序的路径列表。初始化的时候，需要把无向图的路径参数处理成这样的一张表。

```php
/**
 * Class KeLuSiKaErSuanFa [克鲁斯卡尔算法]
 * @package App\ShuJuJieGou
 */
class KeLuSiKaErSuanFa extends WuXiangTuJuZhen
{
    /**
     * @var array [排序过的路径权重列表]
     */
    protected $luJingQuanZhongLieBiao;

    public function __construct($luJingLieBiao)
    {
        $this->luJingQuanZhongLieBiao = [];
        parent::__construct($luJingLieBiao);
    }

    public function gouZaoLinJieJuZhen()
    {
        $this->dingDianShu = 0;
        foreach ($this->luJingLieBiao as $itemLuJing) {
            if (!isset($this->dingDianLieBiao[$itemLuJing[0]])) {
                $this->dingDianLieBiao[$itemLuJing[0]] = $this->dingDianShu;
                $this->zuoBiaoLieBiao[$this->dingDianShu] = $itemLuJing[0];
                ++$this->dingDianShu;
            }
            if (!isset($this->dingDianLieBiao[$itemLuJing[1]])) {
                $this->dingDianLieBiao[$itemLuJing[1]] = $this->dingDianShu;
                $this->zuoBiaoLieBiao[$this->dingDianShu] = $itemLuJing[1];
                ++$this->dingDianShu;
            }
        }
        // 克鲁斯卡尔算法不需要把邻接矩阵构造出来
        $tempLieBiao = $this->luJingLieBiao;
        // 取出列表中权重那一列，作为排序依据
        $paiXuYiJu = array_column($tempLieBiao, 2);
        // 对路径列表按照权重升序排序
        array_multisort($tempLieBiao, SORT_ASC, $paiXuYiJu);
        // 构造排序过的路径权重列表
        foreach ($tempLieBiao as $item) {
            $this->luJingQuanZhongLieBiao[] = [
                $this->dingDianLieBiao[$item[0]],
                $this->dingDianLieBiao[$item[1]],
                $item[2]
            ];
        }
    }
}
```

有了列表之后，从权重最小的路径开始，每次把边和边相关的两个顶点加入最小生成树。这里需要注意的是，在算法执行的过程中会同时存在多个独立的子树，但是最终它们连起来成为一棵树。

在构造最小生成树的过程中，每次循环处理的都是当前有效的权重最小的边。在处理边的时候需要检查两个顶点是不是已经处于任何一棵最小生成树上。

```php
    /**
     * 克鲁斯卡尔算法
     * @return array
     */
    public function keLuSiKaErSuanFa()
    {
        // 记录最小生成树的边
        $zuiXiaoShenChengShu = [];
        // 记录顶点来源，用于判断顶点是不是已经在生成树上了
        $dingDianLaiYuanBiao = [];
        for ($i = 0; $i < $this->dingDianShu; ++$i) {
            $dingDianLaiYuanBiao[$i] = 0;
        }
        for ($i = 0; $i < $this->dingDianShu; ++$i) {
            $m = $this->find($dingDianLaiYuanBiao, $this->luJingQuanZhongLieBiao[$i][0]);
            $n = $this->find($dingDianLaiYuanBiao, $this->luJingQuanZhongLieBiao[$i][1]);
            if ($m !== $n) {
                $dingDianLaiYuanBiao[$m] = $n;
                $zuiXiaoShenChengShu[] = [
                    $this->zuoBiaoLieBiao[$this->luJingQuanZhongLieBiao[$i][0]],
                    $this->zuoBiaoLieBiao[$this->luJingQuanZhongLieBiao[$i][1]]
                ];
            }
        }

        return $zuiXiaoShenChengShu;
    }

    protected function find($dingDianLaiYuanBiao, $k)
    {
        while ($dingDianLaiYuanBiao[$k] !== 0) {
            $k = $dingDianLaiYuanBiao[$k];
        }

        return $k;
    }
```

测试代码：

```php
// $luJingLieBiao = [
//     ['V0', 'V1', 10], ['V0', 'V5', 11],
//     ['V1', 'V2', 18], ['V1', 'V8', 12], ['V1', 'V6', 16],
//     ['V2', 'V3', 22], ['V2', 'V8', 8],
//     ['V3', 'V4', 20], ['V3', 'V6', 24], ['V3', 'V7', 16], ['V3', 'V8', 21],
//     ['V4', 'V5', 26], ['V4', 'V7', 7],
//     ['V5', 'V6', 17],
//     ['V6', 'V7', 19],
// ];
// $wuXiangTu = new KeLuSiKaErSuanFa($luJingLieBiao);
// echo json_encode($wuXiangTu->getLinJieJuZhen());
// echo json_encode($wuXiangTu->keLuSiKaErSuanFa());
```

