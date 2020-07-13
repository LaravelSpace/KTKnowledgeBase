```json
{
  "updated_at": "2020-07-12",
  "updated_by": "KelipuTe",
  "tags": "数据结构,Data Structure,线性表,List,栈,Stack"
}
```

---

## 栈

### 栈的定义

栈（stack）是限定**仅在表尾进行插入和删除操作**的线性表。

我们把允许插入和删除的一端称为栈顶（top），另一端称为栈底（bottom），不含任何数据元素的栈称为空栈。栈又称为**后进先出**（Last In First Out）的线性表，称为 LIFO 结构。你可以想象一下弹夹的结构。

栈的插入操作，称为进栈或者压栈、入栈。栈的删除操作称为出栈或者弹栈。

栈可以使用顺序存储结构，也可以使用链式存储结构，链式存储结构也称为栈链。

### 递归

栈有一个很重要的应用：在应用程序中实现了递归。

我们把一个直接调用自己或通过一系列的调用语句间接调用自己的函数，称做递归函数。

递归函数必须至少有一个条件，满足时递归不再进行，即不在引用自身而是返回值退出。

写递归程序，永远要注意，不要让程序陷入无穷的递归中。

##### PHP 代码

```php
/**
 * Class Zhan [栈]
 * @package App\ShuJuJieGou
 */
class Zhan
{
    /**
     * @var array [栈的数据存储空间]
     */
    protected $zhanKongJian;

    /**
     * @var int [栈顶]
     */
    protected $zhanDing;

    /**
     * @var int [栈底]
     */
    protected $zhanDi;

    public function __construct()
    {
        $this->zhanKongJian = [];
        $this->zhanDing = -1;
        $this->zhanDi = 0;
    }

    /**
     * 获取栈顶元素，但不出栈
     * @return mixed
     */
    public function getZhanDingYuanSu()
    {
        if ($this->zhanDi < $this->zhanDing) return null;
        return $this->zhanKongJian[$this->zhanDing];
    }

    /**
     * 元素入栈
     * @param mixed
     */
    public function ruZhan($item)
    {
        ++$this->zhanDing;
        array_push($this->zhanKongJian, $item);
    }

    /**
     * 元素出栈
     * @return mixed
     */
    public function chuZhan()
    {
        if ($this->zhanDing < $this->zhanDi) return null;
        --$this->zhanDing;
        return array_pop($this->zhanKongJian);
    }
}
```

#### 八皇后问题



#### 后缀（逆波兰）表达式

后缀表达式，也称为逆波兰（Reverse Polish Notation，RPN）表达式。所有的符号都是要在运算的数字后面出现。我们通常写的标准的四则运算表达式也叫中缀表达式。

中缀表达式：$(1-2)*(4+5)$，后缀表达式：`12-45+*`。

中缀表达式：$a+(b-c)*d$ ，后缀表达式：`abc-d*+`。

简单地说，中缀表达式是给人看的，后缀表达式是给计算机看的。

##### PHP 代码

```php
/**
 * 中缀表达式转换后缀表达式
 * @param string $zhongZhuiBiaoDaShi
 * @return string
 */
function getRPN($zhongZhuiBiaoDaShi)
{
    $houZhuiBiaoDaShi = [];

    $operatorStack = new Zhan();

    $numStr = '';
    $strLen = strlen($zhongZhuiBiaoDaShi);
    for ($i = 0; $i < $strLen; ++$i) {
        $itemChar = $zhongZhuiBiaoDaShi[$i];
        // 判断超过9的数字
        if (is_numeric($itemChar)) {
            $numStr .= $itemChar;
            continue;
        } else if ($numStr !== '') {
            $houZhuiBiaoDaShi[] = $numStr;
            $numStr = '';
        }

        switch ($itemChar) {
            case '+':
            case '-':
                // 如果加号和减号前面有其他操作符则需要输出
                while (1) {
                    $operator = $operatorStack->getTopItem();
                    if ($operator === '+' || $operator === '-' || $operator === '*' || $operator === '/') {
                        $operator = $operatorStack->chuZhan();
                        $houZhuiBiaoDaShi[] = $operator;
                    } else {
                        $operatorStack->ruZhan($itemChar);
                        break 2;
                    }
                }
            case '*':
            case '/':
                // 如果乘号和除号前面有乘号或除号则需要输出，加号或减号则不输出
                while (1) {
                    $operator = $operatorStack->getTopItem();
                    if ($operator === '*' && $operator === '/') {
                        $operator = $operatorStack->chuZhan();
                        $houZhuiBiaoDaShi[] = $operator;
                    } else {
                        $operatorStack->ruZhan($itemChar);
                        break 2;
                    }
                }
                break;
            case'(':
                $operatorStack->ruZhan($itemChar);
                break;
            case ')':
                // 遇到右括号时，需要输出操作符，直到匹配到左括号
                while (1) {
                    $operator = $operatorStack->chuZhan();
                    if ($operator === '(') break;
                    $houZhuiBiaoDaShi[] = $operator;
                }
                break;
        }

    }
    // 把最后一个数字输出
    $houZhuiBiaoDaShi[] = $numStr;
    // 把栈中操作符全部输出
    while (1) {
        $operator = $operatorStack->chuZhan();
        if ($operator === null) break;
        $houZhuiBiaoDaShi[] = $operator;
    }

    return implode(' ', $houZhuiBiaoDaShi);
}
```

```php
/**
 * 使用后缀表达式计算结果
 * @param $houZhuiBiaoDaShi
 * @return float
 */
function calculateRPN($houZhuiBiaoDaShi)
{
    $numStack = new Zhan();

    $numStr = '';
    $strLen = strlen($houZhuiBiaoDaShi);
    for ($i = 0; $i < $strLen; ++$i) {
        $itemChar = $houZhuiBiaoDaShi[$i];
        if (is_numeric($itemChar)) {
            $numStr .= $itemChar;
            continue;
        } else if ($itemChar === ' ') {
            if ($numStr !== '') {
                // 字符之间使用空格隔开，用于区分大于9的数字
                $numStack->ruZhan(floatval($numStr));
                $numStr = '';
            }
            continue;
        }
        switch ($itemChar) {
            case '+':
                $num1 = $numStack->chuZhan();
                $num2 = $numStack->chuZhan();
                $num3 = $num2 + $num1;
                $numStack->ruZhan($num3);
                break;
            case '-':
                $num1 = $numStack->chuZhan();
                $num2 = $numStack->chuZhan();
                $num3 = $num2 - $num1;
                $numStack->ruZhan($num3);
                break;
            case '*':
                $num1 = $numStack->chuZhan();
                $num2 = $numStack->chuZhan();
                $num3 = $num2 * $num1;
                $numStack->ruZhan($num3);
                break;
            case '/':
                $num1 = $numStack->chuZhan();
                $num2 = $numStack->chuZhan();
                $num3 = $num2 / $num1;
                $numStack->ruZhan($num3);
                break;
        }

    }
    return $numStack->chuZhan();
}
```

测试代码：

```php
$zhongZhuiBiaoDaShi = '9+(3-1)*3+10/2';
$houZhuiBiaoDaShi = getRPN($zhongZhuiBiaoDaShi);
$jieGuo = calculateRPN($houZhuiBiaoDaShi);

echo json_encode([
    'houZhuiBiaoDaShi' => $houZhuiBiaoDaShi,
    'jieGuo' => $jieGuo,
]);
```

---

## 参考来源

《大话数据结构》，程杰 著，清华大学出版社，2011年6月第1版，2020年3月第24次印刷。
