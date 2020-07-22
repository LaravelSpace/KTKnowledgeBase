```json
{
  "updated_at": "2020-07-11",
  "updated_by": "KelipuTe",
  "tags": "数据结构,Data Structure,树,Tree"
}
```

---

## 二叉树（Binary Tree）

二叉树是树形结构的一个重要类型。在二叉树中，一个元素也称作一个结点。

二叉树是n个有限元素的集合，该集合或者为空、或者由一个称为根（root）的元素及两个不相交的、被分别称为左子树和右子树的二叉树组成，是有序树。当集合为空时，称该二叉树为空二叉树。

二叉树特点是每个结点最多只能有两棵子树，且有左右之分。

### 特殊类型

1、满二叉树：如果一棵二叉树只有度为0的结点和度为2的结点，并且度为0的结点在同一层上，则这棵二叉树为满二叉树。

满二叉树，叶子只能出现在最下一层，非叶子节点的度一定是2。**在同样深度的二叉树中，满二叉树结点数最多**。

2、完全二叉树：深度为k，有n个结点的二叉树当且仅当其每一个结点都与深度为k，有n个结点的满二叉树中编号从1到n的结点一一对应时，称为完全二叉树。

完全二叉树的特点是叶子结点只可能出现在层序最大的两层上，并且某个结点的左分支下子孙的最大层序与右分支下子孙的最大层序相等或大1。层序最大层的叶子结点一定集中在左边的连续位置。如果结点度为1，则该结点只有左孩子。**在同样结点数的二叉树，完全二叉树的深度最小**。

3、斜树：除了根节点外，只有左结点（左斜树）或只有右结点（右斜树）的二叉树。

### 二叉树性质

性质1：二叉树的第i层上至多有$2^{i-1}(i≥1)$个节点。

性质2：深度为h的二叉树中至多含有$2^h-1$个节点。

性质3：若在任意一棵二叉树中，有n个叶子节点，有n2个度为2的节点，则必有n0=n2+1 。

对于任何一棵二叉树，假设叶子结点数为n0，度为1的结点数为n1，度为2的结点数为n2。则$n0+n1+n2=n$。又因为连接（分叉）数总等于总结点数n减1（除根结点外，每个子节点都要连接父结点），得到$n1+2*n2=n-1$。然后，联立方程得出结论。

性质4：具有n个节点的完全二叉树深为$\lfloor\log_2(x)\rfloor+1$（其中x表示不大于n的最大整数，$\lfloor\log_2(x)\rfloor$表示对计算结果向下取整）。

性质5：若对一棵有n个节点的完全二叉树进行顺序编号（1≤i≤n），那么，对于编号为i（i≥1）的节点：

- 当i=1时，该节点为根，它无双亲节点。
- 当i>1时，该节点的双亲节点的编号为i/2 。
- 若2i≤n，则有编号为2的左孩子，否则没有左孩子。
- 若2+1≤n，则有编号为2i+1的右孩子，否则没有右孩子。

### 二叉树遍历

> 1
> /   \
> 2    3
> \      \
> 5     4

前序（先序）遍历（PreOrderTraversal，NLR）：根左右，12534。

中序遍历：左根右（InOrderTraversal，LNR），25134。

后序遍历：左右根（PostOrderTraversal，LRN），52431。

广度优先（层次）遍历（Breadth First Search，BFS）：按层从左到右，12354。

深度优先遍历（Depth First Search，DFS）

已知前序遍历序列和中序遍历序列，可以唯一确定一课二叉树。

已知中序遍历序列和后序遍历序列，可以唯一确定一课二叉树。

已知前序遍历序列和后序遍历序列，**不能**唯一确定一课二叉树。

### 实现二叉树

首先我们要定义二叉树的结点：

```php
/**
 * Class ErChaShuJieDian [二叉树结点]
 * @package App\ShuJuJieGou
 */
class ErChaShuJieDian
{
    /**
     * @var int [结点值]
     */
    public $jieDianZhi;

    /**
     * @var ErChaShuJieDian [左指针]
     */
    public $zuoZhiZhen;

    /**
     * @var ErChaShuJieDian [右指针]
     */
    public $youZhiZhen;

    /**
     * ErChaShuJieDian constructor.
     * @param string $jieDianZhi [结点值]
     */
    public function __construct($jieDianZhi)
    {
        $this->jieDianZhi = '';
        $this->zuoZhiZhen = null;
        $this->youZhiZhen = null;
        if (is_string($jieDianZhi) && $jieDianZhi !== '') {
            $this->jieDianZhi = $jieDianZhi;
        }
    }
}
```

为了方便保存，我们用一个一维数组保存一棵二叉树。首先将二叉树用空结点补全成满二叉树，然后，从上到下，从左到右，依次编号后存入数组对应位置。如上文中的二叉树就可以表示成数组`[1,2,3,null,5,4,null]`，这也是对二叉树进行层次遍历的结果。

同时由于PHP里没有C语言的指针这样的东西，所以我们用一个key-value数组，模拟内存中指针和指针指向的内存空间的关系。在这里指针映射成了数组的下标，指针指向的内存空间就是数组的值。

```php
/**
 * Trait XuNiNeiCunTrait [虚拟内存]
 * @package App\ShuJuJieGou
 */
trait XuNiNeiCunTrait
{
    /**
     * @var array [虚拟内存空间]
     * 用于模拟C语言中的指针
     */
    protected $xuNiNeiCunKongJian;

    /**
     * @var int [虚拟内存大小]
     * 用于记录分配内存的数量
     */
    protected $xuNiNeiCunDaXiao;

    /**
     * 虚拟内存初始化
     */
    protected function xuNiNeiCunChuShiHua()
    {
        $this->xuNiNeiCunKongJian = [];
        $this->xuNiNeiCunDaXiao = 0;
    }

    /**
     * 分配虚拟内存
     * 这个方法需要各个调用的类自行编码
     */
    protected function fenPeiXuNiNeiCun()
    {
    }

    /**
     * 获取虚拟内存数据
     * @param string $zhiZhen
     * @return mixed|null
     */
    protected function huoQuNeiCunShuJu($zhiZhen)
    {
        if (!is_string($zhiZhen) || strlen($zhiZhen) < 1) {
            return null;
        }
        if (isset($this->xuNiNeiCunKongJian[$zhiZhen])) {
            return $this->xuNiNeiCunKongJian[$zhiZhen];
        } else {
            return null;
        }
    }

    /**
     * 整理虚拟内存空间，移除不需要的元素
     * 这个方法需要各个调用的类自行编码
     */
    protected function zhenLiXuNiNeiCunKongJian()
    {
    }
}
```

将数组转换成二叉树的代码如下：

```php
/**
 * Class ErChaShu [二叉树]
 * @package App\ShuJuJieGou
 */
class ErChaShu
{
    use XuNiNeiCunTrait;
    
    /**
     * @var array [二叉树元素表]
     */
    protected $yuanSuBiao;

    /**
     * @var string [根结点指针]
     */
    protected $genJieDianZhiZhen;

    /**
     * ErChaShu constructor.
     * @param array $yuanSuBiao [二叉树元素表]
     * @throws \Exception
     */
    public function __construct($yuanSuBiao)
    {
        $this->xuNiNeiCunChuShiHua();
        $this->yuanSuBiao = [];
        $this->genJieDianZhiZhen = null;
        if (is_array($yuanSuBiao) && count($yuanSuBiao) > 0) {
            $this->yuanSuBiao = $yuanSuBiao;
            $this->gouZaoErChaShu();
            $this->qianXuBianLiXiuJian();
            $this->zhenLiXuNiNeiCunKongJian();
        }
    }
    
    /**
     * 分配虚拟内存
     * @param string $jieDianZhi
     * @return string
     */
    protected function fenPeiXuNiNeiCun($jieDianZhi)
    {
        $zhiZhen = 'zhi_zhen_' . $this->xuNiNeiCunDaXiao;
        ++$this->xuNiNeiCunDaXiao;
        $this->xuNiNeiCunKongJian[$zhiZhen] = new ErChaShuJieDian($jieDianZhi);

        return $zhiZhen;
    }

    /**
     * 整理虚拟内存空间，移除不需要的元素
     */
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
            'gen_jie_dian_zhi_zhen' => $this->genJieDianZhiZhen,
            'xu_ni_nei_cun_kong_jian' => $this->xuNiNeiCunKongJian
        ];
    }

    /**
     * 用二叉树元素表构造二叉树
     * @throws \Exception
     */
    protected function gouZaoErChaShu()
    {
        if (count($this->yuanSuBiao) < 1) {
            return;
        }
        // 创建根结点
        $this->genJieDianZhiZhen = $this->fenPeiXuNiNeiCun($this->yuanSuBiao[0]);
        $biaoChang = count($this->yuanSuBiao);
        for ($i = 1; $i < $biaoChang; $i++) {
            // 依次添加结点
            $jieDianZhiZhen = $this->fenPeiXuNiNeiCun($this->yuanSuBiao[$i]);
            $this->yiCiChaRuJieDian($jieDianZhiZhen);
        }
    }
}
```

插入结点的方法，这里用队列结构实现。在PHP里可以使用数组构造一个简单的队列。使用`array_unshift()`方法和`array_pop()`方法模拟入队和出队。用`array_push()`和`array_shift()`这组方法也可以。

```php
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

                return;
            } else {
                // 如果左指针不为空，则左结点先入队
                array_unshift($duiLie, $tempJieDian->zuoZhiZhen);
            }
            if ($tempJieDian->youZhiZhen === null) {
                // 如果右指针为空就插入结点
                $tempJieDian->youZhiZhen = $jieDianZhiZhen;

                return;
            } else {
                // 如果右指针不为空，则右结点后入队
                array_unshift($duiLie, $tempJieDian->youZhiZhen);
            }
        }
    }
```

在构造完二叉树之后，我们要把填充的空结点剪了，还需要处理虚拟内存空间中无效的元素。

```php
    /**
     * 使用前序遍历修剪多余的空结点
     */
    protected function qianXuBianLiXiuJian()
    {
        $this->qianXuBianLiXiuJianDiGui($this->genJieDianZhiZhen);
    }

    /**
     * @param string $genJieDianZhiZhen
     */
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
            } else {
                $this->qianXuBianLiXiuJianDiGui($genJieDian->zuoZhiZhen);
            }
        }
        if ($genJieDian->youZhiZhen !== null) {
            $youJieDian = $this->huoQuNeiCunShuJu($genJieDian->youZhiZhen);
            if ($youJieDian->jieDianZhi === '') {
                $genJieDian->youZhiZhen = null;
            } else {
                $this->qianXuBianLiXiuJianDiGui($genJieDian->youZhiZhen);
            }
        }
    }
```

前序遍历，中序遍历，后序遍历都采用递归实现。

```php
    /**
     * 前序遍历
     */
    public function qianXuBianLi()
    {
        return $this->qianXuBianLiDiGui($this->genJieDianZhiZhen);
    }

    /**
     * @param string $genJieDianZhiZhen
     * @return string
     */
    protected function qianXuBianLiDiGui($genJieDianZhiZhen)
    {
        if ($genJieDianZhiZhen === null) {
            return '';
        }
        $genJieDian = $this->huoQuNeiCunShuJu($genJieDianZhiZhen);
        if ($genJieDian === null || $genJieDian->jieDianZhi === null) {
            return '';
        }
        $xuLie = '';
        $xuLie .= $genJieDian->jieDianZhi . ';';
        $xuLie .= $this->qianXuBianLiDiGui($genJieDian->zuoZhiZhen);
        $xuLie .= $this->qianXuBianLiDiGui($genJieDian->youZhiZhen);

        return $xuLie;
    }

    /**
     * 中序遍历
     * @return string
     */
    public function zhongXuBianLi()
    {
        return $this->zhongXuBianLiDiGui($this->genJieDianZhiZhen);
    }

    /**
     * @param string $genJieDianZhiZhen
     * @return string
     */
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
        $xuLie .= $this->zhongXuBianLiDiGui($genJieDian->zuoZhiZhen);
        $xuLie .= $genJieDian->jieDianZhi . ';';
        $xuLie .= $this->zhongXuBianLiDiGui($genJieDian->youZhiZhen);

        return $xuLie;
    }

    /**
     * 后序遍历
     */
    public function houXuBianLi()
    {
        return $this->houXuBianLiDiGui($this->genJieDianZhiZhen);
    }

    /**
     * @param string $genJieDianZhiZhen
     * @return string
     */
    protected function houXuBianLiDiGui($genJieDianZhiZhen)
    {
        if ($genJieDianZhiZhen === null) {
            return '';
        }
        $genJieDian = $this->huoQuNeiCunShuJu($genJieDianZhiZhen);
        if ($genJieDian === null || $genJieDian->jieDianZhi === null) {
            return '';
        }
        $xuLie = '';
        $xuLie .= $this->houXuBianLiDiGui($genJieDian->zuoZhiZhen);
        $xuLie .= $this->houXuBianLiDiGui($genJieDian->youZhiZhen);
        $xuLie .= $genJieDian->jieDianZhi . ';';

        return $xuLie;
    }
```

使用递归还可以计算二叉树的最大深度。

```php
    /**
     * 计算二叉树的最大深度
     * @return int
     */
    public function jiSuanZuiDaShenDu()
    {
        return $this->jiSuanZuiDaShenDuDiGui($this->genJieDianZhiZhen);
    }

    /**
     * @param string $genJieDianZhiZhen
     * @return int
     */
    protected function jiSuanZuiDaShenDuDiGui($genJieDianZhiZhen)
    {
        if ($genJieDianZhiZhen === null) {
            return 0;
        }
        $genJieDian = $this->huoQuNeiCunShuJu($genJieDianZhiZhen);
        if ($genJieDian === null || $genJieDian->jieDianZhi === null) {
            return 0;
        }
        $zuoZiShuShenDu = $this->jiSuanZuiDaShenDuDiGui($genJieDian->zuoZhiZhen);
        $youZiShuShenDu = $this->jiSuanZuiDaShenDuDiGui($genJieDian->youZhiZhen);

        return max($zuoZiShuShenDu, $youZiShuShenDu) + 1;
    }
```

二叉树的深度优先遍历算法需要用到栈结构。和队列结构类似，在PHP中栈结构也可以用数组构造一个简单的。使用`array_push()`和`array_pop()`模拟入栈和出栈

```php
    /**
     * 深度优先遍历
     * @return array
     */
    public function shenDuYouXianBianLi()
    {
        $xuLie = [];
        $zhan = [];
        // 根结点入栈
        array_push($zhan, $this->genJieDianZhiZhen);
        while (!empty($zhan)) {
            // 持续遍历，直到栈为空
            // 栈顶元素出栈
            $tempJieDianZhiZhen = array_pop($zhan);
            if ($tempJieDianZhiZhen === null) {
                continue;
            }
            $tempJieDian = $this->huoQuNeiCunShuJu($tempJieDianZhiZhen);
            if ($tempJieDian === null) {
                continue;
            }
            if ($tempJieDian->jieDianZhi !== null) {
                // 存放结点数据
                $xuLie[] = $tempJieDian->jieDianZhi;
            }
            if ($tempJieDian->youZhiZhen !== null) {
                // 右结点先入栈，后遍历
                array_push($zhan, $tempJieDian->youZhiZhen);
            }
            if ($tempJieDian->zuoZhiZhen !== null) {
                // 左结点后入栈，先遍历
                array_push($zhan, $tempJieDian->zuoZhiZhen);
            }
        }

        return $xuLie;
    }
```

二叉树的广度优先遍历算法，需要使用队列实现。

```php
    /**
     * 广度优先遍历
     * @return array
     */
    public function guangDuYouXianBianLi()
    {
        $xuLie = [];
        $duiLie = [];
        array_unshift($duiLie, $this->genJieDianZhiZhen);
        while (!empty($duiLie)) {
            $tempJieDianZhiZhen = array_pop($duiLie);
            if ($tempJieDianZhiZhen === null) {
                continue;
            }
            $tempJieDian = $this->huoQuNeiCunShuJu($tempJieDianZhiZhen);
            if ($tempJieDian === null) {
                continue;
            }
            if ($tempJieDian->jieDianZhi !== null) {
                // 存放结点数据
                $xuLie[] = $tempJieDian->jieDianZhi;
            }
            if ($tempJieDian->zuoZhiZhen !== null) {
                // 左结点先入队，先遍历
                array_unshift($duiLie, $tempJieDian->zuoZhiZhen);
            }
            if ($tempJieDian->youZhiZhen !== null) {
                // 右结点后入队，后遍历
                array_unshift($duiLie, $tempJieDian->youZhiZhen);
            }
        }

        return $xuLie;
    }
```

广度优先遍历算法还可以分层输出结果，我们这里复用上面的代码。如果需要分层输出，则在每一次while循环中，需要计算当前层次要遍历几个结点。然后for循环会把当前层要遍历的结点全部遍历，然后把下一层的结点全部放入队列。同时这个方法这也可以用来计算二叉树的最大深度。

```php
    /**
     * 广度优先遍历，可分层输出结果
     * @return array
     */
    public function guangDuYouXianBianLiFenCeng()
    {
        $xuLie = [];
        $cengShu = 1;
        $duiLie = [];
        array_unshift($duiLie, $this->genJieDianZhiZhen);
        while (!empty($duiLie)) {
            $length = count($duiLie);
            for ($i = 0; $i < $length; $i++) {
                $tempJieDianZhiZhen = array_pop($duiLie);
                if ($tempJieDianZhiZhen === null) {
                    continue;
                }
                $tempJieDian = $this->huoQuNeiCunShuJu($tempJieDianZhiZhen);
                if ($tempJieDian === null) {
                    continue;
                }
                if ($tempJieDian->jieDianZhi !== null) {
                    $xuLie[$cengShu][] = $tempJieDian->jieDianZhi;
                }
                if ($tempJieDian->zuoZhiZhen !== null) {
                    array_unshift($duiLie, $tempJieDian->zuoZhiZhen);
                }
                if ($tempJieDian->youZhiZhen !== null) {
                    array_unshift($duiLie, $tempJieDian->youZhiZhen);
                }
            }
            // 一层遍历结束，层数+1
            $cengShu++;
        }

        return $xuLie;
    }
```

测试代码：

```php
// $yuanSuBiao = ['V0', 'V1', 'V2', null, 'V4', null, 'V6', null, null, 'V9', 'V10', null, null, 'V13', null];
// $erChaShu = new ErChaShu($yuanSuBiao);
// echo json_encode($erChaShu->getErChaShu());
// echo json_encode($erChaShu->qianXuBianLi());
// echo json_encode($erChaShu->zhongXuBianLi());
// echo json_encode($erChaShu->houXuBianLi());
// echo json_encode($erChaShu->jiSuanZuiDaShenDu());
// echo json_encode($erChaShu->guangDuYouXianBianLi());
// echo json_encode($erChaShu->guangDuYouXianBianLiFenCeng());
// echo json_encode($erChaShu->shenDuYouXianBianLi());
```
