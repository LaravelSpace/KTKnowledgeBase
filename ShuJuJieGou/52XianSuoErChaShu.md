```php
{
  "updated_at": "2020-07-13",
  "updated_by": "KelipuTe",
  "tags": "数据结构,Data Structure,树,Tree"
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

首先原先的二叉树的结点结构需要做一些修改，需要额外增加两个整形，用于标识指针域是孩子还是线索。这里假定整形的值：-1=未设置；0=前驱；1=左孩子。

```php
/**
 * Class ShuJieDian [线索二叉树结点]
 * @package App\ShuJuJieGou
 */
class XianSuoErChaShuJieDian extends ErChaShuJieDian
{
    /**
     * @var int [左标识]
     * -1=未设置；0=前驱；1=左孩子
     */
    public $zuoBiaoShi;

    /**
     * @var int [右标识]
     * -1=未设置；0=后继；1=右孩子
     */
    public $youBiaoShi;

    /**
     * ShuJieDian constructor.
     * @param $jieDianZhi [结点值]
     */
    public function __construct($jieDianZhi)
    {
        $this->zuoBiaoShi = -1;
        $this->youBiaoShi = -1;
        parent::__construct($jieDianZhi);
    }
}
```

和之前构造二叉树一样，使用数组的形式构造二叉树，并把多余的空结点剪掉。这里几个复用的函数都需要重写，添加对左右标识属性的支持。

```php
/**
 * Class XianSuoErChaShu [线索二叉树]
 * @package App\ShuJuJieGou
 */
class XianSuoErChaShu extends ErChaShu
{
    /**
     * @var string [前驱结点指针]
     * 前驱结点需要额外记录，而后继结点不需要
     */
    protected $qianQuJieDianZhiZhen;

    /**
     * @var XianSuoErChaShuJieDian [前驱结点]
     * 前驱结点需要额外记录，而后继结点不需要
     */
    protected $qianQuJieDian;

    /**
     * XianSuoErChaShu constructor.
     * @param array $yuanSuBiao [二叉树元素表]
     * @throws \Exception
     */
    public function __construct($yuanSuBiao)
    {
        $this->qianQuJieDianZhiZhen = null;
        $this->qianQuJieDian = null;
        parent::__construct($yuanSuBiao);
    }

    /**
     * @param string $jieDianZhi
     * @return string
     */
    protected function fenPeiXuNiNeiCun($jieDianZhi)
    {
        $zhiZhen = 'zhi_zhen_' . $this->xuNiNeiCunDaXiao;
        ++$this->xuNiNeiCunDaXiao;
        $this->xuNiNeiCunKongJian[$zhiZhen] = new XianSuoErChaShuJieDian($jieDianZhi);

        return $zhiZhen;
    }

    protected function zhenLiXuNiNeiCunKongJian()
    {
        foreach ($this->xuNiNeiCunKongJian as $key => $value) {
            if ($value->jieDianZhi === '') {
                unset($this->xuNiNeiCunKongJian[$key]);
            }
        }
    }

    /**
     * 获取二叉树
     * @return array
     */
    public function getErChaShu()
    {
        return [
            'genJieDianZhiZhen' => $this->genJieDianZhiZhen,
            'xuNiNeiCunKongJian' => $this->xuNiNeiCunKongJian
        ];
    }

    /**
     * 以层次遍历的顺序依次插入结点
     * @param string $jieDianZhiZhen
     * @throws \Exception
     */
    protected function yiCiChaRuJieDian($jieDianZhiZhen)
    {
        // 初始化一个队列
        $duiLie = [];
        // 根结点入队
        array_unshift($duiLie, $this->genJieDianZhiZhen);
        while (!empty($duiLie)) {
            // 持续遍历，直到队列为空
            // 队列元素出队
            $tempJieDianZhiZhen = array_pop($duiLie);
            if ($tempJieDianZhiZhen === null) {
                throw new \Exception("指针：({$tempJieDianZhiZhen})为空");
            }
            $tempJieDian = $this->huoQuNeiCunShuJu($tempJieDianZhiZhen);
            if ($tempJieDian === null) {
                throw new \Exception("结点：({$tempJieDianZhiZhen})为空");
            }
            if ($tempJieDian->zuoZhiZhen === null) {
                // 如果左指针为空就插入结点
                $tempJieDian->zuoZhiZhen = $jieDianZhiZhen;
                $tempJieDian->zuoBiaoShi = 1;

                return;
            } else {
                // 如果左指针不为空，则左结点先入队
                array_unshift($duiLie, $tempJieDian->zuoZhiZhen);
            }
            if ($tempJieDian->youZhiZhen === null) {
                // 如果右指针为空就插入结点
                $tempJieDian->youZhiZhen = $jieDianZhiZhen;
                $tempJieDian->youBiaoShi = 1;

                return;
            } else {
                // 如果右指针不为空，则右结点后入队
                array_unshift($duiLie, $tempJieDian->youZhiZhen);
            }
        }
    }

    protected function qianXuBianLiXiuJianDiGui($genJieDianZhiZhen)
    {
        if ($genJieDianZhiZhen === null) {
            return;
        }
        $genJieDian = $this->huoQuNeiCunShuJu($genJieDianZhiZhen);
        if ($genJieDian === null || $genJieDian->jieDianZhi === null) {
            return;
        }
        if ($genJieDian->zuoZhiZhen !== null) {
            $zuoJieDian = $this->huoQuNeiCunShuJu($genJieDian->zuoZhiZhen);
            if ($zuoJieDian->jieDianZhi === '') {
                $genJieDian->zuoZhiZhen = null;
                $genJieDian->zuoBiaoShi = -1;
            } else {
                $this->qianXuBianLiXiuJianDiGui($genJieDian->zuoZhiZhen);
            }
        }
        if ($genJieDian->youZhiZhen !== null) {
            $youJieDian = $this->huoQuNeiCunShuJu($genJieDian->youZhiZhen);
            if ($youJieDian->jieDianZhi === '') {
                $genJieDian->youZhiZhen = null;
                $genJieDian->youBiaoShi = -1;
            } else {
                $this->qianXuBianLiXiuJianDiGui($genJieDian->youZhiZhen);
            }
        }
    }

    protected function zhongXuBianLiDiGui($genJieDianZhiZhen)
    {
        if ($genJieDianZhiZhen === null) {
            return '';
        }
        $genJieDian = $this->huoQuNeiCunShuJu($genJieDianZhiZhen);
        if ($genJieDian === null || $genJieDian->jieDianZhi === null) {
            return '';
        }
        $xuLie = '';
        if ($genJieDian->zuoBiaoShi === 1) {
            $xuLie .= $this->zhongXuBianLiDiGui($genJieDian->zuoZhiZhen);
        }
        $xuLie .= $genJieDian->jieDianZhi . ';';
        if ($genJieDian->youBiaoShi === 1) {
            $xuLie .= $this->zhongXuBianLiDiGui($genJieDian->youZhiZhen);
        }

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
        $this->qianQuJieDianZhiZhen = null;
        $this->zhongXuBianLiXianSuoHuaDiGui($this->genJieDianZhiZhen);
    }

    /**
     * 中序遍历线索化
     * @param string $genJieDianZhiZhen
     */
    protected function zhongXuBianLiXianSuoHuaDiGui($genJieDianZhiZhen)
    {
        if ($genJieDianZhiZhen === null) {
            return;
        }
        $genJieDian = $this->huoQuNeiCunShuJu($genJieDianZhiZhen);
        if ($genJieDian === null || $genJieDian->jieDianZhi === null) {
            return;
        }
        if ($genJieDian->zuoBiaoShi === 1) {
            // 有左孩子
            $this->zhongXuBianLiXianSuoHuaDiGui($genJieDian->zuoZhiZhen);
        }
        if ($this->qianQuJieDianZhiZhen !== null && $genJieDian->zuoBiaoShi === -1) {
            // 设置前驱线索
            $genJieDian->zuoBiaoShi = 0;
            $genJieDian->zuoZhiZhen = $this->qianQuJieDianZhiZhen;
        }
        if ($this->qianQuJieDian !== null && $this->qianQuJieDian->youBiaoShi === -1) {
            // 设置后继线索
            $this->qianQuJieDian->youBiaoShi = 0;
            $this->qianQuJieDian->youZhiZhen = $genJieDianZhiZhen;
        }
        // 登记自己为前驱
        $this->qianQuJieDianZhiZhen = $genJieDianZhiZhen;
        $this->qianQuJieDian = $genJieDian;

        if ($genJieDian->youBiaoShi === 1) {
            // 有右孩子
            $this->zhongXuBianLiXianSuoHuaDiGui($genJieDian->youZhiZhen);
        }
    }
```

测试代码：

```php
// $yuanSuBiao = ['V0', 'V1', 'V2', null, 'V4', null, 'V6', null, null, 'V9', 'V10', null, null, 'V13', null];
// $xianSuoErChaShu = new XianSuoErChaShu($yuanSuBiao);
// $xianSuoErChaShu->zhongXuBianLiXianSuoHua();
// echo json_encode($xianSuoErChaShu->getErChaShu());
// echo json_encode($xianSuoErChaShu->zhongXuBianLi());
```

