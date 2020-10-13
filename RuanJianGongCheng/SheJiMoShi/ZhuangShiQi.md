```json
{
  "title": "装饰器模式",
  "updated_at": "2020-05-22",
  "updated_by": "KelipuTe",
  "tags": "设计模式,装饰器,Decorator"
}
```

---

## 装饰器模式

这个设计模式由 PHP 编码实现。

装饰器模式（Decorator Pattern）允许向一个现有的对象添加新的功能，同时又不改变其结构。这种类型的设计模式属于结构型模式，它是作为现有的类的一个包装。这种模式创建了一个装饰类，用来包装原有的类，并在保持类方法签名完整性的前提下，提供了额外的功能。

举例，一个人穿衣服，顺序随机，一次一件。从装饰器的角度理解就是，一个人不停地被不同的衣服装饰。

要实现装饰器模式需要被装饰的对象和装饰的对象有同样的用于装饰的接口：

```php
/**
 * Interface DecoratorInterface 定义统一动作的接口
 */
interface DecoratorInterface
{
    public function display();
}
```

下面给出衣服的抽象：

衣服对象接收一个被装饰的对象。

```php
abstract class ClothesAbstract implements DecoratorInterface
{
    protected $component;

    public function __construct(DecoratorInterface $component)
    {
        $this->component = $component;
    }

    public function display()
    {
        $this->component->display();
    }
}
```

下面是几个衣服对象的实例：

这里举例用的都是前置的装饰器，就是后面的装饰器会先用前一个装饰器装饰对象然后再用自己装饰对象。当然也可以有后置的，或者是前后都有的。

```php
class Shirt extends ClothesAbstract
{
    public function display()
    {
        parent::display();
        echo "穿上衬衫。" . PHP_EOL;
    }
}

class Overcoat extends ClothesAbstract
{
    public function display()
    {
        parent::display();
        echo "穿上外套。" . PHP_EOL;
    }
}

class Trousers extends ClothesAbstract
{
    public function display()
    {
        parent::display();
        echo "穿上裤子。" . PHP_EOL;
    }
}

class Shoes extends ClothesAbstract
{
    public function display()
    {
        parent::display();
        echo "穿上鞋子。" . PHP_EOL;
    }
}
```

最后给出人的实例：

```php
class Human implements DecoratorInterface
{
    protected $name;

    public function __construct($name)
    {
        $this->name = $name;
    }

    public function display()
    {
        echo "我是{$this->name}。" . PHP_EOL;
    }
}
```

测试代码：

这里在最后执行 `$tom->display();` 的时候，首先是调用 Shoes 类的 display() 方法。但是调用时会先调用 parent::display() 方法，这里会调用保存在 Shoes 类的 component 参数的 Overcoat 类实例的 display() 方法。依次类推就是一个 Human、Shirt、Trousers、Overcoat、Shoes 的调用顺序。

```php
$tom = new Human('Tom');
$tom = new Shirt($tom);
$tom = new Trousers($tom);
$tom = new Overcoat($tom);
$tom = new Shoes($tom);
$tom->display();
```

---

## 参考

HeadFirst 设计模式

[菜鸟教程 —— 设计模式](https://www.runoob.com/design-pattern/design-pattern-tutorial.html)

