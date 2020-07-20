```json
{
  "updated_at": "2020-07-18",
  "updated_by": "KelipuTe",
  "tags": "数据结构,Data Structure,图,Graph"
}
```

---

## 无向图，邻接矩阵

### 使用邻接矩阵存储无向图

无向图可以用一个列表，通过顶点对，间接表示。无向图这里的顶点对不需要表示顶点的先后顺序。

```php
$luJingLieBiao = [
    ['V0', 'V1'], ['V0', 'V5'],
    ['V1', 'V2'], ['V1', 'V8'], ['V1', 'V6'],
    ['V2', 'V3'], ['V2', 'V8'],
    ['V3', 'V4'], ['V3', 'V6'], ['V3', 'V7'], ['V3', 'V8'],
    ['V4', 'V5'], ['V4', 'V7'],
    ['V5', 'V6'],
    ['V6', 'V7'],
];
```

实际应用中，顶点不一定是连续的数字，而且也不一定是从0开始的，所以需要额外的对顶点做一次映射。比如上面那个列表映射出的顶点列表和对应的坐标列表是：

```json
{"V0":0,"V1":1,"V5":2,"V2":3,"V8":4,"V6":5,"V3":6,"V4":7,"V7":8}
["V0","V1","V5","V2","V8","V6","V3","V4","V7"]
```

最终生成的邻接矩阵是这样的：
$$
\begin{matrix}
X & V0 & V1 & V5 & V2 & V8 & V6 & V3 & V4 & V7 \\
V0 & 0 & 1 & 1 & 0 & 0 & 0 & 0 & 0 & 0 \\
V1 & 1 & 0 & 0 & 1 & 1 & 1 & 0 & 0 & 0 \\
V5 & 1 & 0 & 0 & 0 & 0 & 1 & 0 & 1 & 0 \\
V2 & 0 & 1 & 0 & 0 & 1 & 0 & 1 & 0 & 0 \\
V8 & 0 & 1 & 0 & 1 & 0 & 0 & 1 & 0 & 0 \\
V6 & 0 & 1 & 1 & 0 & 0 & 0 & 1 & 0 & 1 \\
V3 & 0 & 0 & 0 & 1 & 1 & 1 & 0 & 1 & 1 \\
V4 & 0 & 0 & 1 & 0 & 0 & 0 & 1 & 0 & 1 \\
V7 & 0 & 0 & 0 & 0 & 0 & 1 & 1 & 1 & 0
\end{matrix} 
$$

通常我们把纵坐标认为是边的出发点，横坐标认为是边的到达点。图中的 [V0,V1] = 1 就表示，原来的无向图中有 (V0,V1) 这条边。需要注意的是：**邻接矩阵并不是唯一的**，矩阵的最终形态取决于顶点和下标的映射关系。

下面是构造邻接矩阵的PHP代码：

```php
/**
 * Class WuXiangTuJuZhen [无向图，邻接矩阵]
 * @package App\ShuJuJieGou
 */
class WuXiangTuJuZhen
{
    /**
     * @var array [路径列表]
     */
    protected $luJingLieBiao;

    /**
     * @var int [顶点数]
     */
    protected $dingDianShu;

    /**
     * @var array [顶点列表]
     * 顶点列表采用key=>value的形式，key表示顶点，value表示坐标（矩阵坐标）
     */
    protected $dingDianLieBiao;

    /**
     * @var array [坐标列表]
     * 坐标列表保存坐标和顶点的对应关系
     */
    protected $zuoBiaoLieBiao;

    /**
     * @var array [邻接矩阵]
     */
    protected $linJieJuZhen;

    /**
     * @var array [顶点访问列表]
     * 图遍历时保存顶点访问状态
     */
    protected $dingDianFangWenLieBiao;

    /**
     * @var string [遍历结果]
     * 图遍历时保存遍历结果
     */
    protected $bianLiJieGuo;

    /**
     * WuXiangTuJuZhen constructor.
     * @param array $luJingLieBiao
     */
    public function __construct($luJingLieBiao = [])
    {
        $this->luJingLieBiao = [];
        $this->dingDianShu = 0;
        $this->dingDianLieBiao = [];
        $this->zuoBiaoLieBiao = [];
        $this->linJieJuZhen = [];
        $this->dingDianFangWenLieBiao = [];
        $this->bianLiJieGuo = '';

        if (is_array($luJingLieBiao) && count($luJingLieBiao) > 0) {
            $this->luJingLieBiao = $luJingLieBiao;
            $this->gouZaoLinJieJuZhen();
        }
    }
    
    public function getDingDianLieBiao()
    {
        return $this->dingDianLieBiao;
    }

    public function getZuoBiaoLieBiao()
    {
        return $this->zuoBiaoLieBiao;
    }

    public function getLinJieJuZhen()
    {
        return $this->linJieJuZhen;
    }

    /**
     * 构造邻接矩阵
     */
    protected function gouZaoLinJieJuZhen()
    {
        // 遍历统计顶点
        $this->dingDianShu = 0;
        foreach ($this->luJingLieBiao as $itemLuJing) {
            // 这里要统计所有的顶点，然后把顶点映射到坐标上，字符串不适合作为构造矩阵的下标
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
        // 初始化邻接矩阵，全部位置设置为0
        for ($i = 0; $i < $this->dingDianShu; ++$i) {
            for ($j = 0; $j < $this->dingDianShu; ++$j) {
                $this->linJieJuZhen[$i][$j] = 0;
            }
        }
        // 构造邻接矩阵，如果存在边，就把矩阵对应位置置为1，无向图这里矩阵应该是对称的
        foreach ($this->luJingLieBiao as $itemLuJing) {
            $dingDian1 = $itemLuJing[0];
            $dingDian2 = $itemLuJing[1];
            $dingDian1ZuoBiao = $this->dingDianLieBiao[$dingDian1];
            $dingDian2ZuoBiao = $this->dingDianLieBiao[$dingDian2];
            $this->linJieJuZhen[$dingDian1ZuoBiao][$dingDian2ZuoBiao] = 1;
            $this->linJieJuZhen[$dingDian2ZuoBiao][$dingDian1ZuoBiao] = 1;
        }
    }
}
```

### 无向图的遍历

无向图的遍历方式有两种，深度优先遍历和广度优先遍历。

**深度优先遍历**：是指在遍历过程中，一旦发现连通的新的顶点，就移动到新的顶点。并从新的顶点出发继续遍历。新的顶点遍历完了再返回老的顶点继续遍历，直到所有的顶点都被访问到。

```php
    public function getBianLiJieGuo()
    {
        return $this->bianLiJieGuo;
    }

    /**
     * 深度优先遍历
     */
    public function shenDuYouXianBianLi()
    {
        $this->bianLiJieGuo = '';
        // 初始化被访问顶点列表，全部置为未访问
        foreach ($this->zuoBiaoLieBiao as $key => $value) {
            $this->dingDianFangWenLieBiao[$key] = 0;
        }

        for ($i = 0; $i < $this->dingDianShu; ++$i) {
            if ($this->dingDianFangWenLieBiao[$i] === 0)
                // 第一个访问到的顶点是肯定没有被访问过的，所以不用判断连通性
                $this->shenDuYouXianBianLiDiGui($i);
        }

        return $this->bianLiJieGuo;
    }

    protected function shenDuYouXianBianLiDiGui($i)
    {
        // 记录遍历结果，并把顶点加入已访问列表
        $this->bianLiJieGuo .= $this->zuoBiaoLieBiao[$i] . ';';
        $this->dingDianFangWenLieBiao[$i] = 1;
        for ($j = 0; $j < $this->dingDianShu; ++$j) {
            if ($this->linJieJuZhen[$i][$j] === 1 && $this->dingDianFangWenLieBiao[$j] === 0)
      
```

**广度优先遍历**：是指在遍历过程中，先把一个顶点遍历结束，找到所有连通的新的顶点。然后依次遍历这些新的顶点，再找到和这些顶点连通的新的顶点。这里可以借助队列处理顶点先后遍历的顺序。

```php
    /**
     * 广度优先遍历
     * @return string
     */
    public function guangDuYouXianBianLi()
    {
        $this->bianLiJieGuo = '';
        // 初始化被访问顶点列表，全部置为未访问
        foreach ($this->zuoBiaoLieBiao as $key => $value) {
            $this->dingDianFangWenLieBiao[$key] = 0;
        }
        // 广度优先遍历需要用到队列，这里和树的广度优先遍历有点类似
        $duiLie = [];
        for ($i = 0; $i < $this->dingDianShu; ++$i) {
            if ($this->dingDianFangWenLieBiao[$i] === 1) {
                continue;
            }
            $this->bianLiJieGuo .= $this->zuoBiaoLieBiao[$i] . ';';
            $this->dingDianFangWenLieBiao[$i] = 1;
            // 顶点入队
            array_unshift($duiLie, $i);
            while (!empty($duiLie)) {
                // 当队列不为空就持续处理队列元素
                $tempI = array_pop($duiLie);
                for ($j = 0; $j < $this->dingDianShu; ++$j) {
                    // 每一个顶点从头到尾遍历，获取所有连通的顶点并加入队列
                    if ($this->linJieJuZhen[$tempI][$j] === 1 && $this->dingDianFangWenLieBiao[$j] === 0) {
                        $this->bianLiJieGuo .= $this->zuoBiaoLieBiao[$j] . ';';
                        $this->dingDianFangWenLieBiao[$j] = 1;
                        array_unshift($duiLie, $j);
                    }
                }
            }
        }

        return $this->bianLiJieGuo;
    }
```

### 测试代码

```php
$luJingLieBiao = [
    ['V0', 'V1'], ['V0', 'V5'],
    ['V1', 'V2'], ['V1', 'V8'], ['V1', 'V6'],
    ['V2', 'V3'], ['V2', 'V8'],
    ['V3', 'V4'], ['V3', 'V6'], ['V3', 'V7'], ['V3', 'V8'],
    ['V4', 'V5'], ['V4', 'V7'],
    ['V5', 'V6'],
    ['V6', 'V7'],
];
$wuXiangTu = new WuXiangTuJuZhen($luJingLieBiao);
// echo json_encode($wuXiangTu->getDingDianLieBiao());
// echo json_encode($wuXiangTu->getZuoBiaoLieBiao());
echo json_encode($wuXiangTu->getLinJieJuZhen());
// echo json_encode($wuXiangTu->shenDuYouXianBianLi());
// echo json_encode($wuXiangTu->guangDuYouXianBianLi());
```


