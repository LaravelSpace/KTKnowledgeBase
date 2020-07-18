```json
{
  "updated_at": "2020-07-18",
  "updated_by": "KelipuTe",
  "tags": "数据结构,哈夫曼树,Huffman Tree"
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
    }
}
```

然后对二叉树节点稍加改造，增加权重属性。

```php
/**
 * 哈夫曼树结点
 * Class HaFuManShuJieDian
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
     * @param string $jieDianZhi
     * @param int $quanZhong [结点权重]
     */
    public function __construct($jieDianZhi, $quanZhong)
    {
        parent::__construct($jieDianZhi);
        $this->quanZhong = $quanZhong;
    }
}
```

接下来就可以开始实现哈夫曼树了。首先要计算字符串中字符的权重，这是构造哈夫曼树的依据。

```php
/**
 * 哈夫曼树
 * Class HaFuManShu
 * @package App\ShuJuJieGou
 */
class HaFuManShu
{
    /**
     * 临时结点结点值，用于判断是不是有效结点
     */
    protected const TEMP_JIE_DIAN_ZHI = 'temp';

    /**
     * @var string [目标字符串]
     */
    protected $muBiaoZiFuZhuan;

    /**
     * @var array [字符权重列表]
     */
    protected $ziFuQuanZhongMap;

    /**
     * @var array [哈夫曼编码表]
     */
    protected $haFuManBianMaBiao;

    /**
     * @var HaFuManShuJieDian [根结点]
     */
    protected $genJieDian;

    /**
     * HaFuManShu constructor.
     * @param $string [目标字符串]
     */
    public function __construct($string = '')
    {
        $this->muBiaoZiFuZhuan = '';
        $this->ziFuQuanZhongMap = [];
        $this->haFuManBianMaBiao = [];
        $this->genJieDian = null;

        if (!empty($string)) {
            $this->setMuBiaoZiFuChuan($string);
        }
    }

    /**
     * 设置目标字符串并计算字符数目
     * @param string $string
     */
    public function setMuBiaoZiFuChuan($string)
    {
        if(empty($string)) return;
        $this->muBiaoZiFuZhuan = $string;
        $strLen = strlen($string);
        for ($i = 0; $i < $strLen; ++$i) {
            $char = $string[$i];
            // 转义一下特殊字符，用这些特殊字符作为数组键名，不合适。
            if ($char === ' ') $char = 'kong_ge';
            if (isset($this->ziFuQuanZhongMap[$char])) ++$this->ziFuQuanZhongMap[$char];
            else $this->ziFuQuanZhongMap[$char] = 1;
        }
        // 字符权重列表升序排序
        asort($this->ziFuQuanZhongMap);
        $this->gouZaoHaFuManShu();
        $this->gouZaoHaFuManBianMaBiao();
    }

    /**
     * 获取哈夫曼树
     * @return HaFuManShuJieDian
     */
    public function getHaFuManShu()
    {
        return $this->genJieDian;
    }

    /**
     * 获取哈夫曼编码表
     * @return array
     */
    public function getHaFuManBianMaBiao()
    {
        return $this->haFuManBianMaBiao;
    }
}
```

然后就可以使用上面的构造方法，构造哈夫曼树。哈夫曼树是叶子结点开始构造的，构造的过程中需要临时保存这些构造出来的结点。

```php
    /**
     * 构造哈夫曼树
     */
    protected function gouZaoHaFuManShu()
    {
        $ziFuQuanZhongMap = $this->ziFuQuanZhongMap;
        $tempJieDianMap = [];
        $tempJieDianId = 1;
        while (count($ziFuQuanZhongMap) > 1) {
            // 这里获取两个结点的逻辑是一样的，但是不适合做成方法，涉及到数组的删除，有点得不偿失。
            // 获取第一个结点
            reset($ziFuQuanZhongMap);
            $keyZiFu1 = key($ziFuQuanZhongMap);
            $quanZhong1 = $ziFuQuanZhongMap[$keyZiFu1];
            if (isset($tempJieDianMap[$keyZiFu1])) {
                $treeNode1 = $tempJieDianMap[$keyZiFu1];
                unset($ziFuQuanZhongMap[$keyZiFu1]);
                unset($tempJieDianMap[$keyZiFu1]);
            } else {
                $treeNode1 = new HaFuManShuJieDian($keyZiFu1, $quanZhong1);
                unset($ziFuQuanZhongMap[$keyZiFu1]);
            }
            // 获取第二个结点
            // reset($ziFuQuanZhongMap);
            $keyZiFu2 = key($ziFuQuanZhongMap);
            $quanZhong2 = $ziFuQuanZhongMap[$keyZiFu2];
            if (isset($tempJieDianMap[$keyZiFu2])) {
                $treeNode2 = $tempJieDianMap[$keyZiFu2];
                unset($ziFuQuanZhongMap[$keyZiFu2]);
                unset($tempJieDianMap[$keyZiFu2]);
            } else {
                $treeNode2 = new HaFuManShuJieDian($keyZiFu2, $quanZhong2);
                unset($ziFuQuanZhongMap[$keyZiFu2]);
            }
            // 构造上层结点
            $shangCengQuanZhong = $treeNode1->quanZhong + $treeNode2->quanZhong;
            $shangCengTreeNode = new HaFuManShuJieDian(self::TEMP_JIE_DIAN_ZHI, $shangCengQuanZhong);
            if ($treeNode1->quanZhong <= $treeNode2->quanZhong) {
                $shangCengTreeNode->zuoZhiZhen = $treeNode1;
                $shangCengTreeNode->youZhiZhen = $treeNode2;
            } else {
                $shangCengTreeNode->zuoZhiZhen = $treeNode2;
                $shangCengTreeNode->youZhiZhen = $treeNode1;
            }
            // 将构造好的结点放回权重数组并升序排序
            $ziFuQuanZhongMap['temp_' . $tempJieDianId] = $shangCengQuanZhong;
            asort($ziFuQuanZhongMap);
            // 将构造好的结点放到临时列表
            $tempJieDianMap['temp_' . $tempJieDianId] = $shangCengTreeNode;
            // 临时结点ID自增
            ++$tempJieDianId;
        }
        $this->genJieDian = array_shift($tempJieDianMap);
    }
```

有了哈夫曼树，就可以开始构建哈夫曼编码表了，这里使用常规的左子树取0右子树取1的方式构造。遍历方式使用前序遍历，这个很好理解，子树的编码取决于父节点的编码。

```php
    /**
     * 前序遍历构造哈夫曼编码表
     */
    protected function gouZaoHaFuManBianMaBiao()
    {
        $this->gouZaoHaFuManBianMaBiaoDiGui($this->genJieDian);
    }

    /**
     * @param ErChaShuJieDian $genJieDian
     */
    protected function gouZaoHaFuManBianMaBiaoDiGui($genJieDian, $qianZhui = '')
    {
        if ($genJieDian === null) return;
        if ($genJieDian->jieDianZhi === null) return;

        if ($genJieDian->jieDianZhi !== self::TEMP_JIE_DIAN_ZHI)
            $this->haFuManBianMaBiao[$genJieDian->jieDianZhi] = $qianZhui;
        $this->gouZaoHaFuManBianMaBiaoDiGui($genJieDian->zuoZhiZhen, $qianZhui . '0');
        $this->gouZaoHaFuManBianMaBiaoDiGui($genJieDian->youZhiZhen, $qianZhui . '1');
    }
```

有了编码表之后，编码和解吗就比较简单了，单纯的进行字符串匹配就能实现。

```php
    /**
     * 对字符串进行哈夫曼编码
     * @return string
     */
    public function ziFuChuanBianMa()
    {
        $bianMa = '';
        $strLen = strLen($this->muBiaoZiFuZhuan);
        for ($i = 0; $i < $strLen; ++$i) {
            $char = $this->muBiaoZiFuZhuan[$i];
            if ($char === ' ') $char = 'kong_ge';
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
        $haFuManJieMaBiao = array_flip($this->haFuManBianMaBiao);
        $jieMa = '';
        $shiBieMa = '';
        $strLen = strLen($bianMa);
        for ($i = 0; $i < $strLen; ++$i) {
            $shiBieMa .= $bianMa[$i];
            if (isset($haFuManJieMaBiao[$shiBieMa])) {
                if ($haFuManJieMaBiao[$shiBieMa] === 'kong_ge') $jieMa .= ' ';
                else $jieMa .= $haFuManJieMaBiao[$shiBieMa];
                $shiBieMa = '';
            }
        }
        return $jieMa;
    }
```

测试代码。

```php
$string = 'hello world';
$haFuManShu = new HaFuManShu();
$haFuManShu->setMuBiaoZiFuChuan($string);
// echo json_encode($haFuManShu->getHaFuManShu());
// echo json_encode($haFuManShu->getHaFuManBianMaBiao());
$bianMa = $haFuManShu->ziFuChuanBianMa();
echo json_encode($bianMa);
echo json_encode($haFuManShu->ziFuChanJieMa($bianMa));
```

---

## 参考来源

《大话数据结构》，程杰 著，清华大学出版社，2011年6月第1版，2020年3月第24次印刷。

[百度文库--哈夫曼树](https://baike.baidu.com/item/%E5%93%88%E5%A4%AB%E6%9B%BC%E6%A0%91/2305769?fr=aladdin#6)