```json
{
  "title": "观察者模式",
  "updated_at": "2020-05-22",
  "updated_by": "KelipuTe",
  "tags": "设计模式,观察者,Observer"
}
```

---

## 观察者模式

这个设计模式由 PHP 编码实现。

观察者模式定义了对象之间的一对多依赖，当一个对象改变状态时，他的所有依赖者都会收到通知并自动更新。观察者模式提供了一种让主体和观察者之间松耦合的设计，它们依然可以交互，但是不需要清楚彼此的细节，类似于一个触发器。但是要注意观察者模式可以有多组观察者和被观察者参与，最终可能会形成一条长链，这个时候就需要解决流程过长的问题，同时也要注意不要产生回路在成循环调用。

举例，假设有多个气象数据显示屏幕，这些屏幕可能显示同一组或者不同组的气象数据。这里气象数据就是主体是被观察者，显示屏幕是观察者，当显示屏幕收到气象数据改变通知时，会自动更新数据。

下面给出气象数据即被观察者的接口和具体实现代码：

被观察者需要提供注册和移除观察者的接口，就是要知道哪些观察者在观察，因为后续被观察者在发生变化的时候，需要通知观察者做出响应。

```php
/**
 * Interface QiXiangShuJUInterface 气象数据接口，被观察者
 */
interface QiXiangShuJUInterface
{
    // 注册观察者
    public function tianJiaXianShiPingMu(XianShiPingMuInterface $observer);

    // 移除观察者
    public function yiChuXianShiPingMu(XianShiPingMuInterface $observer);

    // 数据发生变化，通知观察者
    public function shuJuFaShengBianHua();
}

/**
 * Class QiXiangShuJu 气象数据主体，被观察者
 */
class QiXiangShuJu implements QiXiangShuJUInterface
{
    // 显示屏幕列表，观察者列表
    protected $xianShiPingMuList;

    // 气温
    protected $qiWen;

    // 湿度
    protected $shiDu;

    // 气压
    protected $qiYa;

    public function __construct()
    {
        $this->xianShiPingMuList = [];
    }

    public function tianJiaXianShiPingMu(XianShiPingMuInterface $xianShiPingMu)
    {
        $this->xianShiPingMuList[$xianShiPingMu->huoQuShiBieHao()] = $xianShiPingMu;
    }

    public function yiChuXianShiPingMu(XianShiPingMuInterface $xianShiPingMu)
    {
        unset($this->xianShiPingMuList[$xianShiPingMu->huoQuShiBieHao()]);
    }

    public function shuJuFaShengBianHua()
    {
        foreach ($this->xianShiPingMuList as $itemXianShiPingMu) {
            $itemXianShiPingMu->shuJuGenXin($this->qiWen, $this->shiDu, $this->qiYa);
        }
    }

    // 设置新的气象数据，通知观察者
    public function setMeasurements(float $qiWen, float $shiDu, float $qiYa)
    {
        $this->qiWen = $qiWen;
        $this->shiDu = $shiDu;
        $this->qiYa = $qiYa;
        $this->shuJuFaShengBianHua();
    }
}
```

显示屏幕即观察者的接口和具体实现代码：

观察者可以保存一份被观察者的实例，用于更新数据时做出相应的处理，也可以没有这个实例直接做出相应的响应，具体视情况而定。

```php
/**
 * Interface XianShiPingMuInterface 显示屏幕接口，观察者
 */
interface XianShiPingMuInterface
{
    // 获取识别号
    public function huoQuShiBieHao();

    // 数据更新时的响应
    public function shuJuGenXin(float $qiWen, float $shiDu, float $qiYa);
}

/**
 * Class XianShiPingMu 显示屏幕，观察者
 */
class XianShiPingMu implements XianShiPingMuInterface
{
    // 观察者唯一标识
    protected $weiYiBiaoShi;

    // 被观察的天气数据主体
    protected $qiXiangShuJu;

    // 气温
    protected $qiWen;

    // 湿度
    protected $shiDu;

    // 气压
    protected $qiYa;

    public function __construct($weiYiBiaoShi, QiXiangShuJu $qiXiangShuJu)
    {
        $this->weiYiBiaoShi = $weiYiBiaoShi;
        $this->qiXiangShuJu = $qiXiangShuJu;
        $qiXiangShuJu->tianJiaXianShiPingMu($this);
    }

    public function huoQuShiBieHao()
    {
        return $this->weiYiBiaoShi;
    }

    public function shuJuGenXin(float $qiWen, float $shiDu, float $qiYa)
    {
        $this->qiWen = $qiWen;
        $this->shiDu = $shiDu;
        $this->qiYa = $qiYa;
        $this->shuJuXianShi();
    }

    // 数据展示
    public function shuJuXianShi()
    {
        echo "显示屏幕：{$this->weiYiBiaoShi}，当前气温：{$this->qiYa};当前湿度：{$this->shiDu};当前气压：{$this->qiYa};" . PHP_EOL;
    }
}
```

测试代码：

这里是两组观察者和被观察者。屏幕 1 和屏幕 3 观察气象数据 1，屏幕 2 观察气象数据 2。在气象数据 1 发生变化时，屏幕 1 和 屏幕 3，会相应地更新数据，而不会影响到屏幕 2。 

```php
$qiXiangShuJu1 = new QiXiangShuJu();
$qiXiangShuJu2 = new QiXiangShuJu();

$xianShiPingMu1 = new XianShiPingMu('屏幕1', $qiXiangShuJu1);
$xianShiPingMu2 = new XianShiPingMu('屏幕2', $qiXiangShuJu2);
$xianShiPingMu3 = new XianShiPingMu('屏幕3', $qiXiangShuJu1);

$qiXiangShuJu1->setMeasurements(1, 11, 111);
$qiXiangShuJu2->setMeasurements(2, 22, 222);
$qiXiangShuJu1->setMeasurements(3, 33, 333);
```

---

## 参考

HeadFirst 设计模式

[菜鸟教程 —— 设计模式](https://www.runoob.com/design-pattern/design-pattern-tutorial.html)