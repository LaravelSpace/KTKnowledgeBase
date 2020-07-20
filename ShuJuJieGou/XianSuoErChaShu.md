```php
{
  "updated_at": "2020-07-13",
  "updated_by": "KelipuTe",
  "tags": "数据结构,Data Structure,树,Tree,二叉树,Binary Tree"
}
```

---

## 线索二叉树

对于n个结点的二叉树，在二叉链存储结构中有n+1个空链域，利用这些空链域存放在某种遍历次序下该结点的前驱结点和后继结点的指针，这些指针称为线索，加上线索的二叉树称为线索二叉树。在二叉树的结点上加上线索的二叉树称为线索二叉树，对二叉树以某种遍历方式（如先序、中序、后序或层次等）进行遍历，使其变为线索二叉树的过程称为对二叉树进行线索化。

二叉树的遍历本质上是将一个复杂的非线性结构转换为线性结构，使每个结点都有了唯一前驱和后继（第一个结点无前驱，最后一个结点无后继）。对于二叉树的一个结点，查找其左右子女是方便的，其前驱后继只有在遍历中得到。线索链表解决了无法直接找到该结点在某种遍历序列中的前驱和后继结点的问题，解决了二叉链表找左、右孩子困难的问题。

线索二叉树的好处是：

- 1、利用线索二叉树进行中序遍历时，不必采用堆栈处理，速度较一般二叉树的遍历速度快，且节约存储空间。
- 2、任意一个结点都能直接找到它的前驱和后继结点。这里注意有左孩子的时候，前驱就是左孩子；有右孩子的时候，后继就是有孩子。

但是插入和删除的时候也需要额外的维护线索，而且子树不能使用主树的线索。

#### 使用中序遍历构造二叉线索树

##### PHP 代码

首先原先的二叉树的结点结构需要做一些修改，需要额外增加两个整形，用于标识指针域是孩子还是线索。这里假定整形的值：-1=未设置；0=前驱；1=左孩子。

```php
/**
 * Class ShuJieDian [线索二叉树结点]
 * @package App\ShuJuJieGou
 */
class XianSuoErChaShuJieDian
{
    /**
     * @var int [结点值]
     */
    public $jieDianZhi;

    /**
     * @var int [左标识]
     * -1=未设置；0=前驱；1=左孩子
     */
    public $zuoBiaoShi;

    /**
     * @var XianSuoErChaShuJieDian [左指针]
     */
    public $zuoZhiZhen;

    /**
     * @var int [右标识]
     * -1=未设置；0=后继；1=右孩子
     */
    public $youBiaoShi;

    /**
     * @var XianSuoErChaShuJieDian [右指针]
     */
    public $youZhiZhen;

    /**
     * ShuJieDian constructor.
     * @param $jieDianZhi [结点值]
     */
    public function __construct($jieDianZhi)
    {
        $this->jieDianZhi = $jieDianZhi;
        $this->zuoZhiZhen = null;
        $this->zuoBiaoShi = -1;
        $this->youZhiZhen = null;
        $this->youBiaoShi = -1;
    }
}
```

和之前构造二叉树一样，使用数组的形式构造二叉树，并把多余的空叶子结点剪掉。

```php
/**
 * Class XianSuoErChaShu [线索二叉树]
 * @package App\ShuJuJieGou
 */
class XianSuoErChaShu
{
    /**
     * @var array [二叉树元素数组]
     */
    protected $yuanSuList;

    /**
     * @var XianSuoErChaShuJieDian [根结点]
     */
    protected $genJieDian;

    /**
     * @var XianSuoErChaShuJieDian [前驱结点]
     * 前驱结点需要额外记录，而后继结点不需要
     */
    protected $qianQuJieDian;

    /**
     * @var XianSuoErChaShuJieDian [前驱结点]
     * 前驱结点需要额外记录，而后继结点不需要
     */
    protected $qianQuJieDianZhi;

    /**
     * ErChaShu constructor.
     * @param $yuanSuList [二叉树元素数组]
     */
    public function __construct($yuanSuList)
    {
        $this->yuanSuList = $yuanSuList;
        $this->genJieDian = null;
        $this->qianQuJieDian = null;
        $this->qianQuJieDianZhi = null;
    }

    /**
     * 用数组构造二叉树
     */
    public function buildTreeWithArray()
    {
        // 空数组返回 null
        if (count($this->yuanSuList) === 0) return null;
        // 创建根结点
        $this->genJieDian = new XianSuoErChaShuJieDian($this->yuanSuList[0]);
        for ($i = 1, $numLen = count($this->yuanSuList); $i < $numLen; $i++) {
            // 依次添加结点
            $jieDian = new XianSuoErChaShuJieDian($this->yuanSuList[$i]);
            $this->chaRuJieDianByNLR($jieDian);
        }
    }

    /**
     * 以先序遍历的顺序插入结点（根左右）
     * @param XianSuoErChaShuJieDian $jieDian
     */
    protected function chaRuJieDianByNLR($jieDian)
    {
        // 初始化一个队列
        $duiLie = [];
        // 根结点入队
        array_unshift($duiLie, $this->genJieDian);
        while (!empty($duiLie)) {
            // 持续遍历结点，直到队列为空
            // 队列元素出队
            $tempJieDian = array_pop($duiLie);
            if ($tempJieDian->zuoZhiZhen === null) {
                // 如果左结点为空就插入结点
                $tempJieDian->zuoZhiZhen = $jieDian;
                $tempJieDian->zuoBiaoShi = 1;

                return;
            } else {
                // 左结点先入队
                array_unshift($duiLie, $tempJieDian->zuoZhiZhen);
            }
            if ($tempJieDian->youZhiZhen === null) {
                // 如果右结点为空就插入结点
                $tempJieDian->youZhiZhen = $jieDian;
                $tempJieDian->youBiaoShi = 1;

                return;
            } else {
                // 右结点后入队
                array_unshift($duiLie, $tempJieDian->youZhiZhen);
            }
        }
    }

    /**
     * 获取二叉树
     * @return XianSuoErChaShuJieDian
     */
    public function getErChaShu()
    {
        return $this->genJieDian;
    }

    /**
     * 使用前序遍历移除多余的空结点
     * @param XianSuoErChaShuJieDian $genJieDian
     */
    public function qianXuBianLiXiuJian($genJieDian)
    {
        if ($genJieDian === null) return;
        if ($genJieDian->jieDianZhi === null) return;
        if ($genJieDian->zuoZhiZhen !== null && $genJieDian->zuoZhiZhen->jieDianZhi === null) {
            $genJieDian->zuoZhiZhen = null;
            $genJieDian->zuoBiaoShi = -1;
        } else {
            $this->qianXuBianLiXiuJian($genJieDian->zuoZhiZhen);
        }
        if ($genJieDian->youZhiZhen !== null && $genJieDian->youZhiZhen->jieDianZhi === null) {
            $genJieDian->youZhiZhen = null;
            $genJieDian->youBiaoShi = -1;
        } else {
            $this->qianXuBianLiXiuJian($genJieDian->youZhiZhen);
        }
    }

    /**
     * 中序遍历
     * @param XianSuoErChaShuJieDian $genJieDian
     * @return string
     */
    public function zhongXuBianLi($genJieDian)
    {
        if ($genJieDian === null) return '';
        if ($genJieDian->jieDianZhi === null) return '';
        $xuLie = '';
        if ($genJieDian->zuoBiaoShi === 1) $xuLie .= $this->zhongXuBianLi($genJieDian->zuoZhiZhen);
        $xuLie .= $genJieDian->jieDianZhi . ';';
        if ($genJieDian->youBiaoShi === 1) $xuLie .= $this->zhongXuBianLi($genJieDian->youZhiZhen);

        return $xuLie;
    }
}
```

使用中序遍历进行线索化。这里要注意逻辑，结合中序遍历左根右的顺序。要先遍历左子树，这个时候全局变量 `$qianQuJieDian` 就会被赋值。当遍历到自己的时候，如果可以设置前驱，那被保存的结点就是自己的前驱。如果前驱可以设置后继，自己就是前驱的后继。然后把自己登记到全局变量里，遍历右子树，这个时候自己就是右子树的前驱。

```php
    /**
     * 中序遍历线索化
     */
    public function zhongXuBianLiXianSuoHua()
    {
        $this->zhongXuBianLiXianSuoHuaDiGui($this->genJieDian);
        $this->qianQuJieDian = null;
    }

    /**
     * 中序遍历线索化
     * @param XianSuoErChaShuJieDian $genJieDian
     */
    protected function zhongXuBianLiXianSuoHuaDiGui($genJieDian)
    {
        if ($genJieDian === null) return;
        if ($genJieDian->jieDianZhi === null) return;
        if ($genJieDian->zuoBiaoShi === 1) {
            // 有左孩子
            $this->zhongXuBianLiXianSuoHuaDiGui($genJieDian->zuoZhiZhen);
        }
        if ($this->qianQuJieDian !== null && $genJieDian->zuoBiaoShi === -1) {
            // 设置前驱线索
            $genJieDian->zuoBiaoShi = 0;
            $genJieDian->zuoZhiZhen = $this->qianQuJieDianZhi;
        }
        if ($this->qianQuJieDian !== null && $this->qianQuJieDian->youBiaoShi === -1) {
            // 设置后继线索
            $this->qianQuJieDian->youBiaoShi = 0;
            $this->qianQuJieDian->youZhiZhen = $genJieDian->jieDianZhi;
        }
        // 登记自己为前驱
        $this->qianQuJieDian = $genJieDian;
        $this->qianQuJieDianZhi = $genJieDian->jieDianZhi;

        if ($genJieDian->youBiaoShi === 1) {
            // 有右孩子
            $this->zhongXuBianLiXianSuoHuaDiGui($genJieDian->youZhiZhen);
        }
    }
```

