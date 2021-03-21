```json
{
  "title": "命令模式",
  "updated_at": "2020-06-03",
  "updated_by": "KelipuTe",
  "tags": "设计模式,命令,Command"
}
```

---

## 命令模式

这个设计模式由 PHP 编码实现。

在软件系统中，行为请求者与行为实现者通常是一种紧耦合的关系。但是某些场合，需要对行为进行记录、撤销或重做时，顺序操作的设计思路显然就不太合适。这时就可以考虑使用命令模式。命令模式是一种数据驱动的设计模式，它属于行为型模式。操作以命令的形式放置在命令对象中，行为请求者通过命令对象调用合适的命令操作行为实现者。 这样就可以将行为请求者与行为实现者解耦，达到可以反复调用命令对象中的命令的目的。

举例，这里用遥控器举例可能比较容易理解，假设需要用一个遥控器同时控制家里的几个电灯和自动门。这种情境下顺序操作的设计虽然可以实现目标，但是一旦电灯或者自动门的内容发生变化，或者想给遥控器增加更多的控制对象的时候就会面临两者都要修改代码的问题。

首先要对命令进行定义：

空指令或无效指令是为了遥控器类初始化的时候赋值使用，防止调用了未定义的命令造成错误。

```php
/**
 * Interface YaoKongQiJieKou 遥控器接口
 */
interface YaoKongQiJieKou
{
    // 执行指令按钮
    public function zhiXingZhiLing();

    // 回退指令按钮
    public function huiTuiZhiLing();
}

/**
 * Class KongZhiLing 空指令或无效指令
 */
class KongZhiLing implements YaoKongQiJieKou
{
    public function zhiXingZhiLing()
    {
    }

    public function huiTuiZhiLing()
    {
    }
}
```

然后分别实现电灯和自动门的实体类和相应的命令类：

设想一下如果命令放在电灯的实体类里面，那么每次想关灯的时候，必须需要有一个 new 一个电灯实例出来，这样就会让遥控器类和电灯类紧耦合。所以中间加一层命令类，这样想关灯的时候，告诉遥控器要关灯就行了，不需要 new 一个电灯实例。而且程序里不同的地方都可以调用同一个遥控器去关灯。

```php
/**
 * Class DianDeng 电灯
 */
class DianDeng
{
    // 电灯位置
    public $weiZhi;

    public function __construct($weiZhi)
    {
        $this->weiZhi = $weiZhi;
    }

    public function daKai()
    {
        echo "打开{$this->weiZhi}的灯" . PHP_EOL;
    }

    public function guanBi()
    {
        echo "关闭{$this->weiZhi}的灯" . PHP_EOL;
    }
}

/**
 * Class KaiDengZhiLing 开灯指令
 */
class KaiDengZhiLing implements YaoKongQiJieKou
{
    /**
     * @var DianDeng
     */
    public $dianDeng;

    public function __construct($dianDeng)
    {
        $this->dianDeng = $dianDeng;
    }

    public function zhiXingZhiLing()
    {
        $this->dianDeng->daKai();
    }

    public function huiTuiZhiLing()
    {
        $this->dianDeng->guanBi();
    }
}

/**
 * Class GuanDengZhiLing 关灯指令
 */
class GuanDengZhiLing implements YaoKongQiJieKou
{
    /**
     * @var DianDeng
     */
    public $dianDeng;

    public function __construct($dianDeng)
    {
        $this->dianDeng = $dianDeng;
    }

    public function zhiXingZhiLing()
    {
        $this->dianDeng->guanBi();
    }

    public function huiTuiZhiLing()
    {
        $this->dianDeng->daKai();
    }
}
```

```php
/**
 * Class ZiDongMen 自动门
 */
class ZiDongMen
{
    // 自动门位置
    public $weiZhi;

    public function __construct($weiZhi)
    {
        $this->weiZhi = $weiZhi;
    }

    public function daKai()
    {
        echo "打开{$this->weiZhi}的自动门" . PHP_EOL;
    }

    public function guanBi()
    {
        echo "关闭{$this->weiZhi}的自动门" . PHP_EOL;
    }
}

class KaiMenZhiLing implements YaoKongQiJieKou
{
    /**
     * @var ZiDongMen
     */
    public $ziDongMen;

    public function __construct($ziDongMen)
    {
        $this->ziDongMen = $ziDongMen;
    }

    public function zhiXingZhiLing()
    {
        $this->ziDongMen->daKai();
    }

    public function huiTuiZhiLing()
    {
        $this->ziDongMen->guanBi();
    }
}

class GuanMenZhiLing implements YaoKongQiJieKou
{
    /**
     * @var ZiDongMen
     */
    public $ziDongMen;

    public function __construct($ziDongMen)
    {
        $this->ziDongMen = $ziDongMen;
    }

    public function zhiXingZhiLing()
    {
        $this->ziDongMen->guanBi();
    }

    public function huiTuiZhiLing()
    {
        $this->ziDongMen->daKai();
    }
}
```

最后是遥控器类：

命令模式的一个特点就是在使用命令之前有一个装载命令的过程。所以遥控器类里面需要有装载命令的逻辑，装载一系列命令和行为实现者。

```php
/**
 * Class YaoKongQi 遥控器
 */
class YaoKongQi
{
    /**
     * @var array [打开指令指令集]
     */
    public $daKaiZhiLingJi;

    /**
     * @var array [关闭按钮指令集]
     */
    public $guanBiZhiLingJi;

    /**
     * 退回指令记录上次的操作
     * @var YaoKongQiJieKou
     */
    public $huiTuiZhiLing;

    /**
     * 设置指令集
     * @param $kongZhiDuiXiang [控制对象]
     * @param $daKaiZhiLing [打开指令]
     * @param $guanBiZhiLing [关闭指令]
     */
    public function sheZhiZhiLingJi($kongZhiDuiXiang, $daKaiZhiLing, $guanBiZhiLing)
    {
        // 初始化空指令
        $this->daKaiZhiLingJi[$kongZhiDuiXiang] = new KongZhiLing();
        $this->guanBiZhiLingJi[$kongZhiDuiXiang] = new KongZhiLing();
        // 如果是有效的指令类型才会存入遥控器的指令集
        if ($daKaiZhiLing instanceof YaoKongQiJieKou) {
            $this->daKaiZhiLingJi[$kongZhiDuiXiang] = $daKaiZhiLing;
        }
        if ($guanBiZhiLing instanceof YaoKongQiJieKou) {
            $this->guanBiZhiLingJi[$kongZhiDuiXiang] = $guanBiZhiLing;
        }
    }

    /**
     * 按下打开按钮
     * @param $kongZhiDuiXiang [控制对象]
     */
    public function daKaiAnNiu($kongZhiDuiXiang)
    {
        $this->daKaiZhiLingJi[$kongZhiDuiXiang]->zhiXingZhiLing();
        $this->huiTuiZhiLing = $this->daKaiZhiLingJi[$kongZhiDuiXiang];
    }

    /**
     * 按下关闭按钮
     * @param $kongZhiDuiXiang [控制对象]
     */
    public function guanBiAnNiu($kongZhiDuiXiang)
    {
        $this->guanBiZhiLingJi[$kongZhiDuiXiang]->zhiXingZhiLing();
        $this->huiTuiZhiLing = $this->guanBiZhiLingJi[$kongZhiDuiXiang];
    }

    /**
     * 回退按钮
     */
    public function huiTuiAnNiu()
    {
        // 直接调用上次操作的回退命令
        $this->huiTuiZhiLing->huiTuiZhiLing();
    }
}
```

测试代码：

命令模式的核心思想可以理解为分层。原本行为请求者是直接调用行为实现者达成目标，但是这样在行为实现者发证变化的时候，不可避免的需要修改行为请求者。命令模式在两者中间加了一个命令调用者，将两者解耦。这样行为请求者只需要知道要调用的命令是什么，而不需要知道行为实现者具体是怎么实现的。这样如果行为实现者或行为实现者其中任何一个发生改动，也只需要改动自己的代码。这样做的后果是增加了程序运行的层级，增加了系统的复杂度，同时系统里多出具体命令类的代码。

```php
$yaoKongQi = new YaoKongQi();

$keTingDianDeng = new DianDeng('客厅');
$keTingKaiDengZhiLing = new KaiDengZhiLing($keTingDianDeng);
$keTingGuanDengZhiLing = new GuanDengZhiLing($keTingDianDeng);
$yaoKongQi->sheZhiZhiLingJi('ke_ting_deng', $keTingKaiDengZhiLing, $keTingGuanDengZhiLing);

$woShiDianDeng = new DianDeng('卧室');
$woShiKaiDengZhiLing = new KaiDengZhiLing($woShiDianDeng);
$woShiGuanDengZhiLing = new GuanDengZhiLing($woShiDianDeng);
$yaoKongQi->sheZhiZhiLingJi('wo_shi', $woShiKaiDengZhiLing, $woShiGuanDengZhiLing);

$qianMenZiDongMen = new ZiDongMen('前门');
$qianMenKaiMenZhiLing = new KaiMenZhiLing($qianMenZiDongMen);
$qianMenGuanMenZhiLing = new GuanMenZhiLing($qianMenZiDongMen);
$yaoKongQi->sheZhiZhiLingJi('qian_men', $qianMenKaiMenZhiLing, $qianMenGuanMenZhiLing);

$yaoKongQi->daKaiAnNiu('wo_shi');
$yaoKongQi->huiTuiAnNiu();
$yaoKongQi->daKaiAnNiu('ke_ting_deng');
$yaoKongQi->guanBiAnNiu('ke_ting_deng');
$yaoKongQi->daKaiAnNiu('qian_men');
$yaoKongQi->guanBiAnNiu('qian_men');
$yaoKongQi->huiTuiAnNiu();
```

## 参考

HeadFirst 设计模式

[菜鸟教程 —— 设计模式](https://www.runoob.com/design-pattern/design-pattern-tutorial.html)