```json
{
  "title": "单例模式",
  "updated_at": "2020-05-29",
  "updated_by": "KelipuTe",
  "tags": "设计模式,工厂,Factory"
}
```

---

## 工厂模式

这个设计模式由 PHP 编码实现。

工厂模式是用工厂对象或工厂方法代替 new 操作的一种模式。工厂模式做的是把实例化具体类的部分抽离出来，使他们不干扰应用的其他部分。虽然这样做会多出额外的工厂模式的代码，但会给系统带来更大的可扩展性和更少的修改量。简而言之就是把实例化具体类和应用代码解耦，但是应用代码会转而和工厂耦合。

工厂顾名思义就是创建产品，根据产品是具体产品还是具体工厂可分为简单工厂模式和工厂方法模式，根据工厂的抽象程度可分为工厂方法模式和抽象工厂模式。这里区分不同的模式其实不是太容易的，三个模式有相互关联的部分，重要的是明白解耦的思想，具体实现实际上顺理成章就出来了。

举例，假设不同的城市有不同的饭馆。不同的饭馆有相同或者不同的菜，这些菜有相同或不同的做法。不同城市的饭馆需要去不同城市的菜市场买菜。这个场景中出现，多对多，这样的复杂组合。这个时候如果在实现类的时候用 new 直接写死创建的对象，那么要饭馆加菜的种类的时候，就避免不了要修改饭馆类。如果一个饭馆要开连锁店，要从当地不同的菜市场进货但是做的菜又是一样的，这个时候如果要加菜，或者修改某一个菜，将会面临倍数修改的情况，也就是每一个饭馆类都要修改。工厂模式可以很好的解决上面这种场景的问题。

下面给出使用工厂模式实现的饭馆、菜和菜市场的实现。

首先是饭馆的抽象，饭馆主要要有个菜谱给客人点菜。抽象出来的工厂提供菜谱和点菜这两个功能。这里可以理解为是一个工厂方法模式，由抽象饭馆的子类决定要实例化菜的具体实例。做出来的菜都会有公共的，买菜，做菜，装盘，上菜的过程，所以针对菜也要做一个抽象。抽象饭馆出来是为了解决多个饭馆实体菜谱不一样的问题，目的是将饭馆和菜解耦。如果只有一个饭馆类，那么会面临所有的菜都放在一个菜谱里，修改起来非常麻烦。

```php
/**
 * Class FanGuan 饭馆
 */
abstract class FanGuan
{
    /**
     * @param $caiMing
     * @return Cai
     */
    abstract public function caiPu($caiMing);

    public function dianCai($caiMing)
    {
        $cai = $this->caiPu($caiMing);

        $cai->maiCai();
        $cai->zuoCai();
        $cai->zhuangPan();
        $cai->shangCai();

        return $cai;
    }
}
```

有了饭馆的抽象下面要有菜的抽象。菜的抽象包括了，原料的来源和做菜的流程。规范做菜的流程是为了在饭馆调用的时候，不需要知道具体是哪个菜在做，只需要知道是菜就行了。菜的原料是要去菜市场买的，菜市场的抽象放在后面描述。菜这里需要一个属性保存菜市场。菜市场可以由外部传入，这样外部不同的饭馆可以用不同的菜市场实体，目的是将菜市场和菜解耦。如果有一家连锁店分别开在江苏和四川，那么同样的菜会使用来自两个地区菜市场的原料。如果，没有菜市场的抽象，在不同地区就会需要代码差别不太大的饭馆实体来对应或者在饭馆里内置菜市场对象，这两种方式都不利于代码扩展。

```php
/**
 * Class Cai 菜
 */
abstract class Cai
{
    /**
     * @var string
     */
    protected $caiMing;

    /**
     * @var CaiShiChang
     */
    protected $caiShiChang;

    public function __construct(CaiShiChang $caiShiChang)
    {
        $this->sheZhiCaiMing();
        $this->caiShiChang = $caiShiChang;
    }

    abstract public function sheZhiCaiMing();

    abstract public function maiCai();

    abstract public function zuoCai();

    public function zhuangPan()
    {
        echo "把{$this->caiMing}装盘" . PHP_EOL;
    }

    public function shangCai()
    {
        echo "把{$this->caiMing}端上桌" . PHP_EOL;
    }
}
```

最后是菜市场的抽象。菜市场的抽象比较简单，只有进货和买菜的功能。进货这里不详细实现，主要实现不同菜市场买不同地区菜的功能。

```php
/**
 * Class CaiShiChang 菜市场
 */
abstract class CaiShiChang
{
    /**
     * @var array
     */
    public $kuCunList;

    public function jinHuo($caiMing, $zhiLiang)
    {
    }

    public function maiCai($caiMing, $zhiLiang)
    {
        echo "买{$zhiLiang}克{$this->kuCunList[$caiMing]['ming_cheng']}" . PHP_EOL;
    }
}
```

两个饭馆实例：

这里 caiPu() 方法，如果从饭馆抽象的角度可以理解成一个抽象方法，但是如果但看 caiPu() 创建菜类的实例时，这又可以理解成是一个简单工厂。就像上文说的，要严格区分几种工厂模式会有些困难，把关注点放在解耦思想上，要容易理解工厂模式的目的。

```php
class JiangSuFanGuan extends FanGuan
{
    protected $caiShiChang;

    public function __construct($caiShiChang = null)
    {
        if ($caiShiChang !== null) {
            $this->caiShiChang = $caiShiChang;
        } else {
            $this->caiShiChang = new JiangSuCaiShiChang();
        }
    }

    public function caiPu($caiMing)
    {
        $caiPu = null;
        if ($caiMing === 'xi_hong_shi_chao_dan') {
            $caiPu = new XiHongShiChaoDan($this->caiShiChang);
        } elseif ($caiMing === 'chao_to_dou_si') {
            $caiPu = new ChaoTuDouSi($this->caiShiChang);
        }

        return $caiPu;
    }
}

class SiChuanFanGuan extends FanGuan
{
    protected $caiShiChang;

    public function __construct($caiShiChang = null)
    {
        if ($caiShiChang !== null) {
            $this->caiShiChang = $caiShiChang;
        } else {
            $this->caiShiChang = new SiChuanCaiShiChang();
        }
    }

    public function caiPu($caiMing)
    {
        $caiPu = null;
        if ($caiMing === 'chao_to_dou_si') {
            $caiPu = new LaChaoTuDouSi($this->caiShiChang);
        }

        return $caiPu;
    }
}
```

三个菜的实例：

菜类中设置菜市场实例的方式其实是依赖注入，从外部注入实例类，减少对类内代码的修改。

```php
class XiHongShiChaoDan extends Cai
{
    public function sheZhiCaiMing()
    {
        $this->caiMing = '西红柿炒鸡蛋';
    }

    public function maiCai()
    {
        $this->caiShiChang->maiCai('xi_hong_shi', 500);
        $this->caiShiChang->maiCai('ji_dan', 200);
    }

    public function zuoCai()
    {
        echo '做西红柿炒鸡蛋' . PHP_EOL;
    }
}

class ChaoTuDouSi extends Cai
{
    public function sheZhiCaiMing()
    {
        $this->caiMing = '炒土豆丝';
    }

    public function maiCai()
    {
        $this->caiShiChang->maiCai('tu_dou', 500);
    }

    public function zuoCai()
    {
        echo '做炒土豆丝' . PHP_EOL;
    }
}

class LaChaoTuDouSi extends ChaoTuDouSi
{
    public function maiCai()
    {
        parent::maiCai();
        $this->caiShiChang->maiCai('la_jiao', 50);
    }

    public function zuoCai()
    {
        parent::zuoCai();
        echo '加点辣椒' . PHP_EOL;
    }
}
```

两个菜市场的实例：

```php
class JiangSuCaiShiChang extends CaiShiChang
{
    public function __construct()
    {
        $this->kuCunList = [
            'xi_hong_shi' => [
                'ming_cheng' => '江苏西红柿',
                'ku_cun'     => 1000
            ],
            'ji_dan'      => [
                'ming_cheng' => '江苏鸡蛋',
                'ku_cun'     => 1000
            ],
            'tu_dou'      => [
                'ming_cheng' => '江苏土豆',
                'ku_cun'     => 1000
            ]
        ];
    }
}

class SiChuanCaiShiChang extends CaiShiChang
{
    public function __construct()
    {
        $this->kuCunList = [
            'tu_dou'  => [
                'ming_cheng' => '四川土豆',
                'ku_cun'     => 1000
            ],
            'la_jiao' => [
                'ming_cheng' => '四川辣椒',
                'ku_cun'     => 1000
            ]
        ];
    }
}
```

测试代码：

从测试代码这里可以看出，工厂模式不需要知道产品是怎么出来的，只需要知道，出来的是什么产品就行了。这样把类的实例化和具体的应用逻辑分开，就可达到解耦的目的。

```php
$fanGuan1 = new JiangSuFanGuan();
$cai1 = $fanGuan1->dianCai('xi_hong_shi_chao_dan');
$cai2 = $fanGuan1->dianCai('chao_to_dou_si');

$fanGuan2 = new SiChuanFanGuan();
$cai3 = $fanGuan2->dianCai('chao_to_dou_si');
```

---

## 参考

HeadFirst 设计模式

[菜鸟教程 —— 设计模式](https://www.runoob.com/design-pattern/design-pattern-tutorial.html)