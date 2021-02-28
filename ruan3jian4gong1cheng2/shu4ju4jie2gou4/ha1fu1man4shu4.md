```json
{
  "updated_at": "2020-07-18",
  "updated_by": "KelipuTe",
  "tags": "数据结构,Data Structure,树,Tree"
}
```

---

## 哈夫曼树

哈夫曼树（霍夫曼树、赫夫曼树）都是指同一个东西。

关于编码的几个概念：

- 定长（Fixed-length）编码：定长仅表明段与段之间长度相同，但没说明是多长，可以定长一字节或者定长二字节。如，ASCII编码。
- 变长（Variable-length）编码：单个编码长度不一致，可以根据整体出现频率调节。如，莫（摩）尔斯电码（Morse Code）。
- 前缀编码：如果在一个编码方案中，任何一个编码都不是其他任何编码的前缀（最左子串），则称该编码是前缀编码。

### 定义

给定N个权值作为N个叶子结点，构造一棵二叉树，若该树的带权路径长度达到最小，称这样的二叉树为最优二叉树，也称为哈夫曼树(Huffman Tree)。哈夫曼树是带权路径长度最短的树，权值较大的结点离根较近。

- 1、路径和路径长度：在一棵树中，从一个结点往下可以达到的孩子或孙子结点之间的通路，称为路径。通路中分支的数目称为路径长度。若规定根结点的层数为1，则从根结点到第L层结点的路径长度为L-1。
- 2、结点的权及带权路径长度：若将树中结点赋给一个有着某种含义的数值，则这个数值称为该结点的权。结点的带权路径长度为：从根结点到该结点之间的路径长度与该结点的权的乘积。
- 3、树的带权路径长度：树的带权路径长度规定为所有叶子结点的带权路径长度之和，记为WPL。

哈夫曼树又称最优二叉树，是一种带权路径长度最短的二叉树。所谓树的带权路径长度，就是树中所有的叶结点的权值乘上其到根结点的路径长度（若根结点为0层，叶结点到根结点的路径长度为叶结点的层数）。树的路径长度是从树根到每一结点的路径长度之和，记为 $WPL=（W1*L1+W2*L2+W3*L3+...+Wn*Ln）$，N个权值Wi（i=1,2,...n）构成一棵有N个叶结点的二叉树，相应的叶结点的路径长度为Li（i=1,2,...n）。可以证明哈夫曼树的WPL是最小的。

### 构造方式

假设有n个权值，则构造出的哈夫曼树有n个叶子结点。 n个权值分别设为 w1、w2、…、wn，则哈夫曼树的构造规则为：

- 1、将w1、w2、…、wn看成是有n棵树的森林(每棵树仅有一个结点)；
- 2、在森林中选出两个根结点的权值最小的树合并，作为一棵新树的左、右子树，且新树的根结点权值为其左、右子树根结点权值之和；
- 3、从森林中删除选取的两棵树，并将新树加入森林；
- 4、重复2、3步，直到森林中只剩一棵树为止，该树即为所求得的哈夫曼树。

哈夫曼树构造出来不一定是唯一的，有可能会有多种形态，但是它们的WPL是相等的。

### 哈夫曼编码

在数据通信中，需要将传送的文字转换成二进制的字符串，用0，1码的不同排列来表示字符。例如，需传送的报文为“AFTER DATA EAR ARE  ART  AREA”，这里用到的字符集为“A，E，R，T，F，D”，各字母出现的次数为{8，4，5，3，1，1}。现要求为这些字母设计编码。要区别6个字母，最简单的二进制编码方式是等长编码，固定采用3位二进制，可分别用000、001、010、011、100、101对“A，E，R，T，F，D”进行编码发送，当对方接收报文时再按照三位一分进行译码。显然编码的长度取决报文中不同字符的个数。若报文中可能出现26个不同字符，则固定编码长度为5。然而，传送报文时总是希望总长度尽可能短。在实际应用中，各个字符的出现频度或使用次数是不相同的，如A、B、C的使用频率远远高于X、Y、Z，自然会想到设计编码时，让使用频率高的用短码，使用频率低的用长码，以优化整个报文编码。

为使不等长编码为前缀编码(即要求一个字符的编码不能是另一个字符编码的前缀)，可用字符集中的每个字符作为叶子结点生成一棵编码二叉树，为了获得传送报文的最短长度，可将每个字符的出现频率作为字符结点的权值赋予该结点上，显然字使用频率越小权值越小，权值越小叶子就越靠下，于是频率小编码长，频率高编码短，这样就保证了此树的最小带权路径长度效果上就是传送报文的最短长度。因此，求传送报文的最短长度问题转化为求由字符集中的所有字符作为叶子结点，由字符出现频率作为其权值所产生的哈夫曼树的问题。利用哈夫曼树来设计二进制的前缀编码，既满足前缀编码的条件，又保证报文编码总长最短。

### PHP 代码

首先哈夫曼树通常使用二叉树实现，我们需要把之前二叉树结点的代码拿过来。

```php
/**
 * 二叉树结点
 * Class ErChaShuJieDian
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
     * @param $jieDianZhi [结点值]
     */
    public function __construct($jieDianZhi)
    {
        $this->jieDianZhi = $jieDianZhi;
        $this->zuoZhiZhen = null;
        $this->youZhiZhen = null;
        if (is_string($jieDianZhi) && $jieDianZhi !== '') {
            $this->jieDianZhi = $jieDianZhi;
        }
    }
}
```

然后对二叉树节点稍加改造，增加权重属性。

```php
/**
 * Class HaFuManShuJieDian [哈夫曼树结点]
 * @package App\ShuJuJieGou
 */
class HaFuManShuJieDian extends ErChaShuJieDian
{
    /**
     * @var int [结点权重]
     */
    public $quanZhong;

    /**
     * HaFuManShuJieDian constructor.
     * @param string $jieDianZhi [结点值]
     * @param int $quanZhong [结点权重]
     */
    public function __construct($jieDianZhi, $quanZhong)
    {
        parent::__construct($jieDianZhi);
        $this->quanZhong = 0;
        if (is_numeric($quanZhong) && $quanZhong > 0) {
            $this->quanZhong = $quanZhong;
        }
    }
}
```

接下来就可以开始实现哈夫曼树了。首先要计算字符串中字符的权重，这是构造哈夫曼树的依据。

```php
/**
 * Class HaFuManShu [哈夫曼树]
 * @package App\ShuJuJieGou
 */
class HaFuManShu
{
    use XuNiNeiCunTrait;

    /**
     * 临时结点结点值，用于判断是不是有效结点
     */
    protected const TEMP_JIE_DIAN_ZHI = 'temp';

    /**
     * @var string [目标字符串]
     */
    protected $muBiaoZiFuChuan;

    /**
     * @var array [字符权重表]
     */
    protected $ziFuQuanZhongBiao;

    /**
     * @var array [哈夫曼编码表]
     */
    protected $haFuManBianMaBiao;

    /**
     * @var string [根结点指针]
     */
    protected $genJieDianZhiZhen;

    /**
     * HaFuManShu constructor.
     * @param string $muBiaoZiFuZhuan [目标字符串]
     */
    public function __construct($muBiaoZiFuZhuan)
    {
        $this->xuNiNeiCunChuShiHua();
        $this->muBiaoZiFuChuan = '';
        $this->ziFuQuanZhongBiao = [];
        $this->haFuManBianMaBiao = [];
        $this->genJieDianZhiZhen = null;
        if (is_string($muBiaoZiFuZhuan) && $muBiaoZiFuZhuan !== '') {
            $this->muBiaoZiFuChuan = $muBiaoZiFuZhuan;
            $this->jiSuanZiFuQuanZhong();
            $this->gouZaoHaFuManShu();
            $this->gouZaoHaFuManBianMaBiao();
        }
    }

    /**
     * 分配虚拟内存
     * @param $ziFu
     * @param $quanZhong
     * @return string
     */
    protected function fenPeiXuNiNeiCun($ziFu, $quanZhong)
    {
        $zhiZhen = 'zhi_zhen_' . $this->xuNiNeiCunDaXiao;
        ++$this->xuNiNeiCunDaXiao;
        $this->xuNiNeiCunKongJian[$zhiZhen] = new HaFuManShuJieDian($ziFu, $quanZhong);

        return $zhiZhen;
    }
    
    /**
     * 获取哈夫曼树
     * @return array
     */
    public function getHaFuManShu()
    {
        return [
            'gen_jie_dian_zhi_zhen' => $this->genJieDianZhiZhen,
            'xu_ni_nei_cun_kong_jian' => $this->xuNiNeiCunKongJian
        ];
    }

    /**
     * 计算字符串中字符的权重
     */
    protected function jiSuanZiFuQuanZhong()
    {
        $chuanChang = strlen($this->muBiaoZiFuChuan);
        for ($i = 0; $i < $chuanChang; ++$i) {
            $char = $this->muBiaoZiFuChuan[$i];
            // 转义一下特殊字符，用这些特殊字符作为数组键名，不合适。
            if ($char === ' ') {
                $char = 'kong_ge';
            }
            if (isset($this->ziFuQuanZhongBiao[$char])) {
                ++$this->ziFuQuanZhongBiao[$char];
            } else {
                $this->ziFuQuanZhongBiao[$char] = 1;
            }
        }
        // 字符权重表按升序排序
        asort($this->ziFuQuanZhongBiao);
    }
}
```

然后就可以使用上面的构造方法，构造哈夫曼树。哈夫曼树是从叶子结点开始构造的，构造的过程中需要临时保存这些构造出来的结点。

```php
    /**
     * 构造哈夫曼树
     */
    protected function gouZaoHaFuManShu()
    {
        $ziFuQuanZhongBiao = $this->ziFuQuanZhongBiao;
        while (count($ziFuQuanZhongBiao) > 1) {
            // 这里获取两个结点的逻辑是一样的，但是不适合做成方法，涉及到数组的删除，有点得不偿失。
            // 获取第一个结点
            reset($ziFuQuanZhongBiao);
            $keyZiFu1 = key($ziFuQuanZhongBiao);
            $quanZhong1 = $ziFuQuanZhongBiao[$keyZiFu1];
            if (isset($this->xuNiNeiCunKongJian[$keyZiFu1])) {
                $jieDianZhiZhen1 = $keyZiFu1;
                unset($ziFuQuanZhongBiao[$keyZiFu1]);
            } else {
                $jieDianZhiZhen1 = $this->fenPeiXuNiNeiCun($keyZiFu1, $quanZhong1);
                unset($ziFuQuanZhongBiao[$keyZiFu1]);
            }
            $jieDian1 = $this->huoQuNeiCunShuJu($jieDianZhiZhen1);
            // 获取第二个结点
            $keyZiFu2 = key($ziFuQuanZhongBiao);
            $quanZhong2 = $ziFuQuanZhongBiao[$keyZiFu2];
            if (isset($this->xuNiNeiCunKongJian[$keyZiFu2])) {
                $jieDianZhiZhen2 = $keyZiFu2;
                unset($ziFuQuanZhongBiao[$keyZiFu2]);
            } else {
                $jieDianZhiZhen2 = $this->fenPeiXuNiNeiCun($keyZiFu2, $quanZhong2);
                unset($ziFuQuanZhongBiao[$keyZiFu2]);
            }
            $jieDian2 = $this->huoQuNeiCunShuJu($jieDianZhiZhen2);
            // 构造上层结点
            $shangCengQuanZhong = $jieDian1->quanZhong + $jieDian2->quanZhong;
            $shangCengZhiZhen = $this->fenPeiXuNiNeiCun(self::TEMP_JIE_DIAN_ZHI, $shangCengQuanZhong);
            $shangcengJieDian = $this->huoQuNeiCunShuJu($shangCengZhiZhen);
            if ($jieDian1->quanZhong <= $jieDian2->quanZhong) {
                $shangcengJieDian->zuoZhiZhen = $jieDianZhiZhen1;
                $shangcengJieDian->youZhiZhen = $jieDianZhiZhen2;
            } else {
                $shangcengJieDian->zuoZhiZhen = $jieDianZhiZhen2;
                $shangcengJieDian->youZhiZhen = $jieDianZhiZhen1;
            }
            // 将构造好的结点放回权重数组并升序排序
            $ziFuQuanZhongBiao[$shangCengZhiZhen] = $shangCengQuanZhong;
            asort($ziFuQuanZhongBiao);
        }
        reset($ziFuQuanZhongBiao);
        $this->genJieDianZhiZhen = key($ziFuQuanZhongBiao);
    }
```

有了哈夫曼树，就可以开始构建哈夫曼编码表了，这里使用常规的左子树取0右子树取1的方式构造。遍历方式使用前序遍历，这个很好理解，子树的编码取决于父节点的编码。

```php
    /**
     * 获取哈夫曼编码表
     * @return array
     */
    public function getHaFuManBianMaBiao()
    {
        return $this->haFuManBianMaBiao;
    }

    /**
     * 前序遍历构造哈夫曼编码表
     */
    protected function gouZaoHaFuManBianMaBiao()
    {
        $this->gouZaoHaFuManBianMaBiaoDiGui($this->genJieDianZhiZhen);
    }

    /**
     * @param string $genJieDianZhiZhen
     * @param string $qianZhui
     */
    protected function gouZaoHaFuManBianMaBiaoDiGui($genJieDianZhiZhen, $qianZhui = '')
    {
        if ($genJieDianZhiZhen === null) {
            return;
        }
        $genJieDian = $this->huoQuNeiCunShuJu($genJieDianZhiZhen);
        if ($genJieDian->jieDianZhi === null) {
            return;
        }
        if ($genJieDian->jieDianZhi !== self::TEMP_JIE_DIAN_ZHI) {
            $this->haFuManBianMaBiao[$genJieDian->jieDianZhi] = $qianZhui;
        }
        $this->gouZaoHaFuManBianMaBiaoDiGui($genJieDian->zuoZhiZhen, $qianZhui . '0');
        $this->gouZaoHaFuManBianMaBiaoDiGui($genJieDian->youZhiZhen, $qianZhui . '1');
    }
```

有了编码表之后，编码和解吗就比较简单了，单纯的进行字符串匹配就能实现。

```php
    /**
     * 对目标字符串进行哈夫曼编码
     * @return string
     */
    public function ziFuChuanBianMa()
    {
        $bianMa = '';
        $chuanChang = strLen($this->muBiaoZiFuChuan);
        for ($i = 0; $i < $chuanChang; ++$i) {
            $char = $this->muBiaoZiFuChuan[$i];
            if ($char === ' ') {
                $char = 'kong_ge';
            }
            $bianMa .= $this->haFuManBianMaBiao[$char];
        }

        return $bianMa;
    }

    /**
     * 将哈夫曼编码进行解码
     * @param $bianMa [哈夫曼编码串]
     * @return string
     */
    public function ziFuChanJieMa($bianMa)
    {
        $jieMa = '';
        $shiBieMa = '';
        $haFuManJieMaBiao = array_flip($this->haFuManBianMaBiao);
        $chuanChang = strLen($bianMa);
        for ($i = 0; $i < $chuanChang; ++$i) {
            $shiBieMa .= $bianMa[$i];
            if (isset($haFuManJieMaBiao[$shiBieMa])) {
                if ($haFuManJieMaBiao[$shiBieMa] === 'kong_ge') {
                    $jieMa .= ' ';
                } else {
                    $jieMa .= $haFuManJieMaBiao[$shiBieMa];
                }
                // 匹配到一次之后，要把当前这个识别码置空，进行下一轮的匹配
                $shiBieMa = '';
            }
        }

        return $jieMa;
    }
}
```

测试代码：

```php
// $muBiaoZiFuZhuan = 'hello world';
// $haFuManShu = new HaFuManShu($muBiaoZiFuZhuan);
// echo json_encode($haFuManShu->getHaFuManShu());
// echo json_encode($haFuManShu->getHaFuManBianMaBiao());
// $bianMa = $haFuManShu->ziFuChuanBianMa();
// echo json_encode($bianMa);
// echo json_encode($haFuManShu->ziFuChanJieMa($bianMa));
```



