```json
{
  "updated_at": "2020-07-12",
  "updated_by": "KelipuTe",
  "tags": "数据结构,Data Structure,树,Tree"
}
```

---

## 树

树（Tree）是$n(n\geq0)$个结点的**有限集**。n=0时称为空树。在任意一棵非空树中：

- 1、有且仅有一个特定的称为根（Root）的结点；
- 2、当$n>1$时，其余结点可分为$m(m>0)$个互不相交的有限集T1、T2、......、Tm，其中每一个集合本身又是一棵树，并且称为根的子树（SubTree）。

### 相关概念

- 结点：包含一个数据元素及若干指向子树分支的信息。
- 结点的度：一个结点拥有子树的数目称为结点的度（Degree）。
- 叶子结点（Leaf）：也称为终端结点，没有子树的结点或者度为零的结点。
- 分支结点：也称为非终端结点、内部节点，度不为零的结点称为非终端结点。
- 树的度：树中所有结点的度的最大值。
- 结点的层次：从根结点开始，假设根结点为第1层，根结点的孩子结点为第2层，依此类推，如果某一个结点位于第L层，则其孩子结点位于第L+1层。
- 树的深度（Depth）：也称为树的高度，树中所有结点的层次最大值称为树的深度。
- 有序树：如果树中各棵子树的次序是有先后次序，则称该树为有序树。
- 无序树：如果树中各棵子树的次序没有先后次序，则称该树为无序树。
- 森林（Forest）：由$m(m≥0)$棵互不相交的树构成一片森林。如果把一棵非空的树的根结点删除，则该树就变成了一片森林，森林中的树由原来根结点的各棵子树构成。

### 树的存储结构

双亲表示法：以一组连续空间存储树的结点。在每个结点中，附设一个指示器指示其双亲结点在数组中的位置。

多重链表表示法：每个结点有多个指针域，其中每个指针指向一棵子树的根结点。这种方法有两种方案：

- 1、指针域的个数就等于树的度，这种方法会造成空间浪费。
- 2、每个结点的指针域的个数等于结点的度，额外取一个位置来存储指针域的个数，这种方法克服了空间浪费的问题，但在运算上会带来时间上的损耗。

孩子表示法：把每个结点的孩子排列起来，以单链表作存储结构，则n个结点有n个孩子链表，如果是叶子结点则此单链表为空。然后n个头指针又组成一个线性表，采用顺序存储结构，存放进一个一维数组中。这个结构的问题是不知道某个结点的双亲。

双亲表示法可以和孩子表示法结合，可以弥补各自的劣势。

孩子兄弟表示法：任意一棵树，它的结点的第一个孩子如果存在就是唯一的，它的右兄弟如果存在也是唯一的。因此，我们设置两个指针，分别指向该结点的第一个孩子和该结点的右兄弟。

### 树、二叉树、森林的转换

树转换为二叉树：

- 1、加线：在所有兄弟结点之间加一条线。
- 2、去线：对树中每一个结点，只保留它与第一个孩子结点的连线。
- 3、层次调整：调整一下，让图看起来顺眼一点。

二叉树转换为树：

- 1、加线：若某节点的左孩子结点存在，则将这个左孩子的右孩子节点、右孩子的右孩子结点、右孩子的右孩子的右孩子结点。。。等。将该结点和这些右孩子结点连起来。
- 2、去线：删除原二叉树中所有节点与其右孩子的连线。
- 3、层次调整：调整一下，让图看起来顺眼一点。

森林转换为二叉树：

- 1、把每一棵树转换为二叉树。
- 2、第一棵二叉树不动，从第二棵开始，依次把后一棵二叉树的根结点作为前一棵二叉树的右孩子。

二叉树转换成森林：

- 1、从根结点开始，如果右孩子存在，则删除与右孩子之间的连线。分离出的二叉树，如果根结点依然存在右孩子，则重复上面的步骤。
- 2、将分离出的二叉树转换成树。

### 树的遍历

树的遍历方式分两种：

- 先根遍历：先访问根结点，然后依次先根遍历根的每一棵子树。
- 后根遍历：先依次后根遍历每一棵子树，然后再访问根节点。

森林的遍历和树的遍历类似，也分两种：先序遍历和后序遍历。

### 树的用途

编译系统的语法树，操作系统的目录管理、文件管理，数据库的索引，网络系统的域名管理。

### 实现树结构

首先需要一个树节点，树结点记录结点信息，父结点和子结点列表。这里和二叉树是不一样的，树结构可以有n个子结点。

```php
/**
 * Class ShuJieDian [树节点]
 * @package App\ShuJuJieGou
 */
class ShuJieDian
{
    /**
     * @var int [结点值]
     */
    public $jieDianZhi;

    /**
     * @var ErChaShuJieDian [父指针]
     */
    public $fuZhiZhen;

    /**
     * @var array [孩子指针数组]
     */
    public $zhiZhenYu;

    /**
     * ShuJieDian constructor.
     * @param string $jieDianZhi [结点值]
     */
    public function __construct($jieDianZhi)
    {
        $this->jieDianZhi = '';
        $this->fuZhiZhen = null;
        $this->zhiZhenYu = [];
        if (is_numeric($jieDianZhi) || (is_string($jieDianZhi) && $jieDianZhi !== '')) {
            $this->jieDianZhi = $jieDianZhi;
        }
    }
}
```

然后我们就可以实现树结构了。首先需要一个结点关系表保存各个节点之间的关系。

在构造树的过程中，依然需要使用二叉树那里的虚拟内存的方式模拟指针，详细的可以看二叉树那里的内容。和二叉树不一样的是，树结构的构造可以不用按一定的顺序。二叉树那里是从根节点开始，一个一个构造整棵二叉树的。但是树结构这里可以先构造分叉，最后在合并成一棵完整的树结构。

所以在构造结点的时候如果发现父结点还没有构造，那就要额外的构造父结点。如果构造结点的时候发现结点已经存在了，那就说明子结点在父节点之前构造了，但是子结点并不知道父结点的结点值，所以需要更新子结点构造出来的父结点的结点值。

```php
/**
 * Class Shu [树]
 * @package App\ShuJuJieGou
 */
class Shu
{
    use XuNiNeiCunTrait;

    /**
     * 临时结点结点值，用于判断是不是有效结点
     */
    protected const TEMP_JIE_DIAN_ZHI = 'temp';

    /**
     * @var array [关系表]
     */
    protected $guanXiBiao;

    /**
     * @var string [结点id和虚拟内存指针对照表]
     */
    protected $idHeZhiZhenDuiZhaoBiao;

    /**
     * @var string [根结点指针]
     */
    protected $genJieDianZhiZhen;

    /**
     * Shu constructor.
     * @param array $guanXiBiao
     */
    public function __construct($guanXiBiao = [])
    {
        $this->xuNiNeiCunChuShiHua();
        $this->guanXiBiao = [];
        $this->idHeZhiZhenDuiZhaoBiao = [];
        $this->genJieDianZhiZhen = null;
        if (is_array($guanXiBiao) && count($guanXiBiao) > 0) {
            $this->guanXiBiao = $guanXiBiao;
            $this->gouZaoShu();
        }
    }

    /**
     * @param string $jieDianZhi
     * @return string
     */
    protected function fenPeiXuNiNeiCun($jieDianZhi)
    {
        $zhiZhen = 'zhi_zhen_' . $this->xuNiNeiCunDaXiao;
        ++$this->xuNiNeiCunDaXiao;
        $this->xuNiNeiCunKongJian[$zhiZhen] = new ShuJieDian($jieDianZhi);

        return $zhiZhen;
    }

    /**
     * 构造树
     */
    protected function gouZaoShu()
    {
        if (count($this->guanXiBiao) < 1) {
            return;
        }
        foreach ($this->guanXiBiao as $key => $item) {
            if (isset($this->idHeZhiZhenDuiZhaoBiao[$key])) {
                // 如果结点已经创建了，说明是子结点创建的临时结点
                $jieDianZhiZhen = $this->idHeZhiZhenDuiZhaoBiao[$key];
                $jieDian = $this->huoQuNeiCunShuJu($jieDianZhiZhen);
                $jieDian->jieDianZhi = $item['jie_dian_zhi'];
            } else {
                $jieDianZhiZhen = $this->fenPeiXuNiNeiCun($item['jie_dian_zhi']);
                $jieDian = $this->huoQuNeiCunShuJu($jieDianZhiZhen);
                $this->idHeZhiZhenDuiZhaoBiao[$key] = $jieDianZhiZhen;
            }
            if ($item['fu_zhi_zhen'] === null) {
                // 没有父结点，那这个结点就是根结点
                $this->genJieDianZhiZhen = $jieDianZhiZhen;
            } else {
                // 有父节点就要把这个结点连到父结点
                $fuJieDianId = $item['fu_zhi_zhen'];
                if (isset($this->idHeZhiZhenDuiZhaoBiao[$fuJieDianId])) {
                    $fuJieDianZhiZhen = $this->idHeZhiZhenDuiZhaoBiao[$fuJieDianId];
                    $fuJieDian = $this->huoQuNeiCunShuJu($fuJieDianZhiZhen);
                    $fuJieDian->zhiZhenYu[] = $jieDianZhiZhen;
                } else {
                    $fuJieDianZhiZhen = $this->fenPeiXuNiNeiCun(self::TEMP_JIE_DIAN_ZHI);
                    $fuJieDian = $this->huoQuNeiCunShuJu($fuJieDianZhiZhen);
                    $fuJieDian->zhiZhenYu[] = $jieDianZhiZhen;
                    $this->idHeZhiZhenDuiZhaoBiao[$fuJieDianId] = $fuJieDian;
                }
                $jieDian->fuZhiZhen = $fuJieDianZhiZhen;
            }
        }
    }

    /**
     * 获取树
     * @return array
     */
    public function getShu()
    {
        return [
            'genJieDianZhiZhen' => $this->genJieDianZhiZhen,
            'idHeZhiZhenDuiZhaoBiao' => $this->idHeZhiZhenDuiZhaoBiao,
            'xuNiNeiCunKongJian' => $this->xuNiNeiCunKongJian
        ];
    }
}
```

测试代码。

```php
// $guanXiBiao = [
//     'id01' => ['jie_dian_zhi' => 1, 'fu_zhi_zhen' => null],
//     'id02' => ['jie_dian_zhi' => 2, 'fu_zhi_zhen' => 'id01'],
//     'id03' => ['jie_dian_zhi' => 3, 'fu_zhi_zhen' => 'id01'],
//     'id04' => ['jie_dian_zhi' => 4, 'fu_zhi_zhen' => 'id02'],
//     'id05' => ['jie_dian_zhi' => 5, 'fu_zhi_zhen' => 'id02'],
//     'id06' => ['jie_dian_zhi' => 6, 'fu_zhi_zhen' => 'id02'],
//     'id07' => ['jie_dian_zhi' => 7, 'fu_zhi_zhen' => 'id03'],
// ];
// $shu = new Shu($guanXiBiao);
// echo json_encode($shu->getShu());
```

