```json
{
  "updated_at": "2020-07-12",
  "updated_by": "KelipuTe",
  "tags": "数据结构,Data Structure,线性表,List,栈,Stack"
}
```

---

## 栈

### 定义

栈（stack）是限定**仅在表尾进行插入和删除操作**的线性表。

我们把允许插入和删除的一端称为栈顶（top），另一端称为栈底（bottom），不含任何数据元素的栈称为空栈。栈又称为**后进先出**（Last In First Out）的线性表，称为 LIFO 结构。你可以想象一下弹夹的结构。

栈的插入操作，称为进栈或者压栈、入栈。栈的删除操作称为出栈或者弹栈。

栈可以使用顺序存储结构，也可以使用链式存储结构，链式存储结构也称为栈链。

### 递归

栈有一个很重要的应用：在应用程序中实现了递归。我们把一个直接调用自己或通过一系列的调用语句间接调用自己的函数，称做递归函数。

递归函数必须至少有一个条件，满足时递归不再进行，即不在引用自身而是返回值退出。写递归程序，永远要注意，不要让程序陷入无穷的递归中。

关于递归的调用，我觉得不应该把递归的入口直接暴露给外部，这样风险很大。建议将递归入口封装，做好递归前的各项校验工作，保证递归可以顺利的执行结束。

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
        if ($this->zhanDing < $this->zhanDi) {
            return null;
        }

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
        if ($this->zhanDing < $this->zhanDi) {
            return null;
        }
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

中缀表达式转后缀表达式的时候，数字是直接输出的，操作符需要入栈，下面是一个比较简单的例子，有很多细节没有考虑到。

- 括号的优先级是最高的，需要用到一点贪心算法的策略。一旦遇到括号就要一直出栈操作符直到遇见另一半括号。
- 乘号和除号优先级比加号和减号要高，乘号和除号在入栈时如果前面是加号和减号，则直接入栈，如果是乘号和除号则需要先把前面的操作符出栈，自己在入栈。
- 加号和减号入栈时要输出前面所有的符号。

```php
/**
 * Class HouZhuiBiaoDaShi [后缀表达式]
 * @package App\ShuJuJieGou
 */
class HouZhuiBiaoDaShi
{
    /**
     * @var string [中缀表达式]
     */
    protected $zhongZhuiBiaoDaShi;

    /**
     * @var string [后缀表达式]
     */
    protected $houZhuiBiaoDaShi;

    /**
     * HouZhuiBiaoDaShi constructor.
     * @param $zhongZhuiBiaoDaShi [中缀表达式]
     */
    public function __construct($zhongZhuiBiaoDaShi)
    {
        $this->zhongZhuiBiaoDaShi = '';
        $this->houZhuiBiaoDaShi = '';
        if (is_string($zhongZhuiBiaoDaShi) && $zhongZhuiBiaoDaShi !== '') {
            $this->zhongZhuiBiaoDaShi = $zhongZhuiBiaoDaShi;
            $this->zhongZhuiZhuanHouZhui();
        }
    }

    /**
     * 获取后缀表达式
     * @return string
     */
    public function getHouZhuiBiaoDaShi()
    {
        return $this->houZhuiBiaoDaShi;
    }

    /**
     * 中缀表达式转后缀表达式
     */
    protected function zhongZhuiZhuanHouZhui()
    {
        $houZhuiBiaoDaShi = [];
        $caoZuoFuZhan = new Zhan();
        $shuZiString = '';
        $chuanChang = strlen($this->zhongZhuiBiaoDaShi);
        for ($i = 0; $i < $chuanChang; ++$i) {
            $itemChar = $this->zhongZhuiBiaoDaShi[$i];
            // 判断超过9的数字
            if (is_numeric($itemChar)) {
                $shuZiString .= $itemChar;

                continue;
            } else if ($shuZiString !== '') {
                $houZhuiBiaoDaShi[] = $shuZiString;
                $shuZiString = '';
            }
            // 分别判断各个操作符
            switch ($itemChar) {
                case '+':
                case '-':
                    // 如果加号和减号前面有其他操作符则需要输出
                    while (1) {
                        $caoZuoFu = $caoZuoFuZhan->getZhanDingYuanSu();
                        if ($caoZuoFu === '+' || $caoZuoFu === '-' || $caoZuoFu === '*' || $caoZuoFu === '/') {
                            $caoZuoFu = $caoZuoFuZhan->chuZhan();
                            $houZhuiBiaoDaShi[] = $caoZuoFu;
                        } else {
                            $caoZuoFuZhan->ruZhan($itemChar);
                            break 2;
                        }
                    }
                    break;
                case '*':
                case '/':
                    // 如果乘号和除号前面有乘号或除号则需要输出，加号或减号则不输出
                    while (1) {
                        $caoZuoFu = $caoZuoFuZhan->getZhanDingYuanSu();
                        if ($caoZuoFu === '*' && $caoZuoFu === '/') {
                            $caoZuoFu = $caoZuoFuZhan->chuZhan();
                            $houZhuiBiaoDaShi[] = $caoZuoFu;
                        } else {
                            $caoZuoFuZhan->ruZhan($itemChar);
                            break 2;
                        }
                    }
                    break;
                case'(':
                    $caoZuoFuZhan->ruZhan($itemChar);
                    break;
                case ')':
                    // 遇到右括号时，需要输出操作符，直到匹配到左括号
                    while (1) {
                        $caoZuoFu = $caoZuoFuZhan->chuZhan();
                        if ($caoZuoFu === '(') {
                            break;
                        }
                        $houZhuiBiaoDaShi[] = $caoZuoFu;
                    }
                    break;
            }
        }
        // 把最后一个数字输出
        $houZhuiBiaoDaShi[] = $shuZiString;
        // 把栈中操作符全部输出
        while (1) {
            $caoZuoFu = $caoZuoFuZhan->chuZhan();
            if ($caoZuoFu === null) {
                break;
            }
            $houZhuiBiaoDaShi[] = $caoZuoFu;
        }
        $this->houZhuiBiaoDaShi = implode(' ', $houZhuiBiaoDaShi);
    }
}
```

使用后缀表达式计算结果的时候就比较简单了，碰到数字就入栈，碰到操作符就出栈两个数字，计算之后的结果再入栈。最后栈里的那个数字就是表达式计算的结果。

```php
    /**
     * 使用后缀表达式计算结果
     * @return float
     */
    public function jiSuanHouZhui()
    {
        $shuZiZhan = new Zhan();
        $shuZiString = '';
        $chuanChang = strlen($this->houZhuiBiaoDaShi);
        for ($i = 0; $i < $chuanChang; ++$i) {
            $itemChar = $this->houZhuiBiaoDaShi[$i];
            if (is_numeric($itemChar)) {
                $shuZiString .= $itemChar;

                continue;
            } else if ($itemChar === ' ') {
                if ($shuZiString !== '') {
                    // 字符之间使用空格隔开，用于区分大于9的数字
                    $shuZiZhan->ruZhan(floatval($shuZiString));
                    $shuZiString = '';
                }

                continue;
            }
            switch ($itemChar) {
                case '+':
                    $num1 = $shuZiZhan->chuZhan();
                    $num2 = $shuZiZhan->chuZhan();
                    $num3 = $num2 + $num1;
                    $shuZiZhan->ruZhan($num3);
                    break;
                case '-':
                    $num1 = $shuZiZhan->chuZhan();
                    $num2 = $shuZiZhan->chuZhan();
                    $num3 = $num2 - $num1;
                    $shuZiZhan->ruZhan($num3);
                    break;
                case '*':
                    $num1 = $shuZiZhan->chuZhan();
                    $num2 = $shuZiZhan->chuZhan();
                    $num3 = $num2 * $num1;
                    $shuZiZhan->ruZhan($num3);
                    break;
                case '/':
                    $num1 = $shuZiZhan->chuZhan();
                    $num2 = $shuZiZhan->chuZhan();
                    $num3 = $num2 / $num1;
                    $shuZiZhan->ruZhan($num3);
                    break;
            }
        }

        return $shuZiZhan->chuZhan();
    }
```

测试代码：

```php
// $zhongZhuiBiaoDaShi = '9+(3-1)*3+10/2';
// $houZhuiBiaoDaShi = new HouZhuiBiaoDaShi($zhongZhuiBiaoDaShi);
// $result = [
//     'zhongZhuiBiaoDaShi' => $zhongZhuiBiaoDaShi,
//     'houZhuiBiaoDaShi' => $houZhuiBiaoDaShi->getHouZhuiBiaoDaShi(),
//     'jieGuo' => $houZhuiBiaoDaShi->jiSuanHouZhui(),
// ];
// echo json_encode($result);
```

