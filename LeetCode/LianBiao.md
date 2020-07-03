```json
{
  "updated_by": "KelipuTe",
  "updated_at": "2020-06-19",
  "tags": "数据结构,二叉树"
}
```

---

## 链表

这个数据结构由 PHP 编码实现。

首先是链表节点的定义：

```php
/**
 * Definition for a singly-linked list.
 */
class ListNode
{
    public $val = 0;
    public $next = null;

    function __construct($val = 0, $next = null)
    {
        $this->val = $val;
        $this->next = $next;
    }
}
```

用数组构造链表（顺序、倒序）。

```php
/**
 * 数组构造链表（顺序）
 * @param $numList
 * @return ListNode
 */
function makeListAsc($numList)
{
    if (empty($numList)) return null;
    $head = new ListNode($numList[0]);
    $tail = $head;
    for ($i = 1; $i < count($numList); $i++) {
        $node = new ListNode($numList[$i]);
        $tail->next = $node;
        $tail = $node;
    }

    return $head;
}
```

```php
/**
 * 数组构造链表（倒序）
 * @param $numList
 * @return ListNode
 */
function makeListDesc($numList)
{
    if (empty($numList)) return null;
    $length = count($numList);
    $root = new ListNode($numList[$length - 1]);
    for ($i = $length - 2; $i >= 0; $i--) {
        $root = new ListNode($numList[$i], $root);
    }

    return $root;
}
```

数字直接构造链表，需要先把数字拆了。

```php
/**
 * 数字构造链表
 * @param $num
 * @return ListNode|null
 */
function makeList($num)
{
    if ($num === 0) return new ListNode($num);
    $listHead = null;
    $listTail = null;
    do {
        // 数字每次对 10 取余，获得最后一位
        $listVal = $num % 10;
        $listNode = new ListNode($listVal);
        if ($listHead === null) {
            $listHead = $listNode;
            $listTail = $listNode;
        } else {
            $listTail->next = $listNode;
            $listTail = $listNode;
        }
        $num = (int)(($num - $listVal) / 10);
    } while ($num > 0);

    return $listHead;
}
```