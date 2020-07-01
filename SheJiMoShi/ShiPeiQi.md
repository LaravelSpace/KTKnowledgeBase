```json
{
  "title": "观察者模式",
  "updated_at": "2020-06-04",
  "updated_by": "KelipuTe",
  "tags": "设计模式,适配器,Adapter"
}
```

---

## 适配器模式

这个设计模式由 PHP 编码实现。

适配器模式是作为两个不兼容的接口之间的桥梁，将一个类的接口转换成希望的另外一个接口。适配器模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作，提高复用性，比较灵活。但是适配器会使得系统不易从整体进行把握，因此如果不是很有必要，不要使用适配器而是进行重构。适配器还有一个使用场景就是需要修改一个正在正常运行的接口，这个时候可以考虑使用适配器。可以在源代码不修改的情况下添加新的功能。

举例，三脚插头没法插在四孔插座上，这时就需要一个适配器插在四孔插座上，把它变成三孔插座。

下面是适配器的代码：

```php
class SanJiaoChaTou
{
    protected $mingCheng;

    public function __construct($mingCheng)
    {
        $this->mingCheng = '三脚插头 ' . $mingCheng;
    }

    public function chaRuChaZuo()
    {
        echo "{$this->mingCheng} 插入插座。" . PHP_EOL;
    }
}

class SanKongChaZuo
{
    public function sanKong220V()
    {
        echo '三孔 220 伏插座。' . PHP_EOL;
    }
}

class SiKongChaZuo
{
    public function siKong380V()
    {
        echo '四孔 380 伏插座。' . PHP_EOL;
    }
}

class SiZhuanSanShiPeiQi
{
    /**
     * @var SiKongChaZuo
     */
    protected $siKongChaZuo;

    public function __construct($siKongChaZuo)
    {
        $this->siKongChaZuo = $siKongChaZuo;
    }

    public function sanKong220V()
    {
        $this->siKongChaZuo->siKong380V();
        echo '转换插孔，电压适配。' . PHP_EOL;
    }
}
```

测试代码：

当一个三孔插头想使用三孔插座时，直接使用就可以了。但是如果给的是一个四孔插座，这时候就需要一个适配器去连接三孔插头和四孔插座，适配器内部适配四孔插座，并对外提供一个三孔插座给三孔插头使用。

```php
$sanJiaoChaTou = new SanJiaoChaTou('插头1');
$sanKongChaZuo = new SanKongChaZuo();
$sanKongChaZuo->sanKong220V();
$sanJiaoChaTou->chaRuChaZuo();

$sanJiaoChaTou = new SanJiaoChaTou('插头2');
$siKongChaZuo = new SiKongChaZuo();
$siZhuanSanShiPeiQi = new SiZhuanSanShiPeiQi($siKongChaZuo);
$siZhuanSanShiPeiQi->sanKong220V();
$sanJiaoChaTou->chaRuChaZuo();
```

---

## 参考

HeadFirst 设计模式

[菜鸟教程 —— 设计模式](https://www.runoob.com/design-pattern/design-pattern-tutorial.html)