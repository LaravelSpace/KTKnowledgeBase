```json
{
"updated_at": "2020-07-11",
"updated_by": "KelipuTe",
"tags": "数据结构,Data Structure,树,Tree,二叉树,Binary Tree"
}
```

---

## 二叉树

二叉树（Binary Tree）是树形结构的一个重要类型。

### 定义

二叉树是n个有限元素的集合，该集合或者为空、或者由一个称为根（root）的元素及两个不相交的、被分别称为左子树和右子树的二叉树组成，是有序树。当集合为空时，称该二叉树为空二叉树。在二叉树中，一个元素也称作一个结点。二叉树特点是每个结点最多只能有两棵子树，且有左右之分。

### 相关术语

- 结点：包含一个数据元素及若干指向子树分支的信息。
- 结点的度：一个结点拥有子树的数目称为结点的度。
- 叶子结点：也称为终端结点，没有子树的结点或者度为零的结点。
- 分支结点：也称为非终端结点，度不为零的结点称为非终端结点。
- 树的度：树中所有结点的度的最大值。
- 结点的层次：从根结点开始，假设根结点为第1层，根结点的孩子结点为第2层，依此类推，如果某一个结点位于第L层，则其孩子结点位于第L+1层。
- 树的深度：也称为树的高度，树中所有结点的层次最大值称为树的深度。
- 有序树：如果树中各棵子树的次序是有先后次序，则称该树为有序树。
- 无序树：如果树中各棵子树的次序没有先后次序，则称该树为无序树。
- 森林：由m（m≥0）棵互不相交的树构成一片森林。如果把一棵非空的树的根结点删除，则该树就变成了一片森林，森林中的树由原来根结点的各棵子树构成。

### 特殊类型

1、满二叉树：如果一棵二叉树只有度为0的结点和度为2的结点，并且度为0的结点在同一层上，则这棵二叉树为满二叉树。

满二叉树，叶子只能出现在最下一层，非叶子节点的度一定是2。**在同样深度的二叉树中，满二叉树结点数最多**。

2、完全二叉树：深度为k，有n个结点的二叉树当且仅当其每一个结点都与深度为k，有n个结点的满二叉树中编号从1到n的结点一一对应时，称为完全二叉树。

完全二叉树的特点是叶子结点只可能出现在层序最大的两层上，并且某个结点的左分支下子孙的最大层序与右分支下子孙的最大层序相等或大1。层序最大层的叶子结点一定集中在左边的连续位置。如果结点度为1，则该结点只有左孩子。**在同样结点数的二叉树，完全二叉树的深度最小**。

3、斜树：除了根节点外，只有左结点或只有右节点的二叉树。

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

>       1
>   /   \
>2    3
>   \      \
>      5     4

前序（先序）遍历（Preorder Traversal，NLR）：根左右，12534。

中序遍历：左根右（Inorder Traversal，LNR），25134。

后序遍历：左右根（Postorder Traversal，LRN），52431。

广度优先（层次）遍历（Breadth First Search，BFS）：按层从左到右，12354。

深度优先遍历（Depth First Search，DFS）

### PHP 代码

树节点的定义

```php
/**
 * 定义结点
 */
class TreeNode
{
    /**
     * @var int 结点值
     */
    public $value = null;

    /**
     * @var TreeNode 左子树
     */
    public $left = null;

    /**
     * @var TreeNode 右子树
     */
    public $right = null;

    public function __construct($value)
    {
        $this->value = $value;
    }
}
```

为了方便保存，我们用一个一维数组保存一棵二叉树。首先将二叉树用空结点补全成满二叉树，然后，从上到下，从左到右，依次编号后存入数组对应位置。如上文中的二叉树就可以表示成数组`[1,2,3,null,5,4,null]`，这也是二对叉树进行前序遍历的结果。将数组转换成二叉树的代码如下。

```php
/**
 * 用数组构造二叉树
 * @param array $numList
 * @return TreeNode
 */
function buildTreeWithArray($numList)
{
    // 空数组返回 null
    if (count($numList) === 0) return null;
    // 创建根结点
    $root = new TreeNode($numList[0]);
    for ($i = 1, $numLen = count($numList); $i < $numLen; $i++) {
        // 依次添加结点
        $node = new TreeNode($numList[$i]);
        insertNode($root, $node);
    }

    return $root;
}
```

插入结点的方法，这里用队列实现。

```php
/**
 * 以先序遍历的顺序插入结点（根左右）
 * @param TreeNode $root
 * @param TreeNode $iNode
 * @return TreeNode
 */
function insertNodeByNLR($root, $iNode)
{
    $queue = [];
    // 根结点入队
    array_unshift($queue, $root);
    while (!empty($queue)) {
        // 持续遍历结点，直到队列为空
        // 队列元素出队
        $tNode = array_pop($queue);
        if ($tNode->left === null) {
            // 如果左结点为空就插入结点
            $tNode->left = $iNode;

            return $root;
        } else {
            // 左结点先入队
            array_unshift($queue, $tNode->left);
        }
        if ($tNode->right === null) {
            // 如果右结点为空就插入结点
            $tNode->right = $iNode;

            return $root;
        } else {
            // 右结点后入队
            array_unshift($queue, $tNode->right);
        }
    }
}
```

前序遍历，中序遍历，后序遍历都采用递归实现。

```php
/**
 * 前序遍历
 * @param TreeNode $root
 */
function NLRTraverse($root)
{
    if ($root === null) return;
    if ($root->value === null) return;
    echo $root->value . ';';
    NLRTraverse($root->left);
    NLRTraverse($root->right);
}

/**
 * 中序遍历
 * @param TreeNode $root
 */
function LNRTraverse($root)
{
    if ($root === null) return;
    if ($root->value === null) return;
    LNRTraverse($root->left);
    echo $root->value . ';';
    LNRTraverse($root->right);
}

/**
 * 后序遍历
 * @param TreeNode $root
 */
function LRNTraverse($root)
{
    if ($root === null) return;
    if ($root->value === null) return;
    LRNTraverse($root->left);
    LRNTraverse($root->right);
    echo $root->value . ';';
}
```

使用递归还可以计算二叉树的最大深度。

```php
/**
 * 计算二叉树的最大深度
 * @param TreeNode $root
 * @return int
 */
function maxDepth($root)
{
    if ($root === null) return 0;
    if ($root->value === null) return 0;
    $left = maxDepth($root->left);
    $right = maxDepth($root->right);

    return max($left, $right) + 1;
}
```

二叉树的广度优先算法，使用队列实现。

```php
/**
 * 广度优先遍历
 * @param $root
 * @return array
 */
function BFSTraverse($root)
{
    $nodeList = [];

    $queue = [];
    // 根结点入队
    array_unshift($queue, $root);
    while (!empty($queue)) {
        // 持续输出结点，直到队列为空
        // 队列元素出队
        $tNode = array_pop($queue);
        // 存放结点数据
        if ($tNode->value !== null) $nodeList[] = $tNode->value;
        // 左结点先入队，先遍历
        if ($tNode->left !== null) array_unshift($queue, $tNode->left);
        // 右结点后入队，后遍历
        if ($tNode->right !== null) array_unshift($queue, $tNode->right);
    }

    return $nodeList;
}
```

广度优先算法分层输出，还是使用队列实现，每一次while循环计算当前层要遍历几个结点，然后for循环会把当前层要遍历的结点全部遍历然后把下一层的结点全部放入队列。这也可以用来计算二叉树的最大深度。

```php
/**
 * 广度优先遍历，可分层输出结果
 * @param TreeNode $root
 * @return array
 */
function BFSTraverse2($root)
{
    $nodeList = [];
    $level = 1;

    $queue = [];
    // 根结点入队
    array_push($queue, $root);
    while ($length = count($queue)) {
        // 持续输出结点，直到队列为空
        for ($i = 0; $i < $length; $i++) {
            // 队列元素出队
            $tNode = array_shift($queue);
            if ($tNode->value !== null) $nodeList[$level][] = $tNode->value;
            // 左结点先入队，先遍历
            if ($tNode->left !== null) array_push($queue, $tNode->left);
            // 右结点后入队，后遍历
            if ($tNode->right !== null) array_push($queue, $tNode->right);
        }
        // 一层遍历结束，层数+1
        $level++;
    }

    return $nodeList;
}
```

除了广度优先遍历还可以使用深度优先遍历，与广度优先遍历使用队列不同的是，深度优先遍历要用到栈结构。

```php
/**
 * 深度优先遍历
 * @param TreeNode $root
 * @return array
 */
function DFSTraverse($root)
{
    $nodeList = [];

    $stack = [];
    // 根结点入栈
    array_push($stack, $root);
    while (!empty($stack)) {
        // 持续输出结点，直到栈为空
        // 栈顶元素出栈
        $tNode = array_pop($stack);
        // 存放结点数据
        if ($tNode->value !== null) $nodeList[] = $tNode->value;
        // 右结点先入栈，后遍历
        if ($tNode->right !== null) array_push($stack, $tNode->right);
        // 左结点后入栈，先遍历
        if ($tNode->left !== null) array_push($stack, $tNode->left);
    }

    return $nodeList;
}
```

---

## 参考

[百度文库--二叉树](https://baike.baidu.com/item/%E4%BA%8C%E5%8F%89%E6%A0%91)