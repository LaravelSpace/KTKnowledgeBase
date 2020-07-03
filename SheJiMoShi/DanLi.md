```json
{
  "title": "单例模式",
  "updated_at": "2020-06-01",
  "updated_by": "KelipuTe",
  "tags": "设计模式,单例,Singleton"
}
```

---

## 单例模式

这个设计模式由 PHP 编码实现。

单例模式可以确保一个类只有**唯一一个实例**，并为这个类提供一个全局访问点。当有想控制实例数目或者想要一个全局唯一计数器（序列号）之类的需求时，就可以使用单例模式。单例模式可以避免频繁的创建和销毁实例或者创建很多资源开销很大的实例。但是单例模式没有扩展性，因为单例不能继承。

举例，假设现在只有一个豆浆机，所以每次使用的时候，不管这个豆浆机此前被何人在何时用过，这次拿到的豆浆机也应该还是同一个。所以不管是加料还是把做好的豆浆倒出来，都是同一个豆浆机的容积在发生变化。

下面给出豆浆机的代码：

```php
// 豆浆机单例类
class DouJiangJi
{
    // 用静态私有变量保存该类唯一实例
    static private $weiYiDouJiangJi;

    // 豆浆机容量
    protected $rongLiang;

    // 防止使用 new 直接创建对象
    private function __construct()
    {
        $this->rongLiang = 500;
    }

    // 防止使用 clone() 克隆对象
    private function __clone()
    {
    }

    // 对外提供静态方法获取唯一实例对象
    static public function huoQuDouJiangJi()
    {
        if (!self::$weiYiDouJiangJi instanceof self) {
            // 如果对象没有创建过，则创建对象
            self::$weiYiDouJiangJi = new self();
        }
        return self::$weiYiDouJiangJi;
    }

    // 加配料，不能超过豆浆机容积
    public function jiaPeiLiao($peiLiao)
    {
        if ($this->rongLiang < $peiLiao) {
            echo "豆浆机装不下 {$peiLiao}ml 配料了。" . PHP_EOL;
        } else {
            echo "装入 {$peiLiao}ml 配料。" . PHP_EOL;
            $this->rongLiang -= $peiLiao;
        }
    }

    // 做豆浆，会消耗掉所有的配料
    public function zuoDouJiang()
    {
        $douJiang = 500 - $this->rongLiang;
        echo "做 {$douJiang}ml 豆浆。" . PHP_EOL;
        $this->rongLiang = 500;
        echo "把做好的豆浆全部倒出。" . PHP_EOL;
    }
}
```

测试代码如下：

```php
$douJiangJi1 = DouJiangJi::huoQuDouJiangJi();
$douJiangJi2 = DouJiangJi::huoQuDouJiangJi();

$douJiangJi1->jiaPeiLiao(200);
$douJiangJi2->jiaPeiLiao(200);
$douJiangJi1->jiaPeiLiao(200);
$douJiangJi1->zuoDouJiang();

$douJiangJi2->jiaPeiLiao(500);
$douJiangJi2->zuoDouJiang();
```

因为这里用了单例模式所以两个变量都获取到了唯一的一个豆浆机对象，所以加料的时候不管是哪一个加料，最终容积的变化都会反映在同一个豆浆机实例上。由于代码里把容积设置成了 500ml。所以下面的测试代码会在第 6 行，判断出容量不够。

---

## 参考

HeadFirst 设计模式

[菜鸟教程 —— 设计模式](https://www.runoob.com/design-pattern/design-pattern-tutorial.html)