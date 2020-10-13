```json
{
  "title": "外观模式（Facade）",
  "updated_at": "2020-05-21",
  "updated_by": "KelipuTe",
  "tags": "设计模式,外观,Facade"
}
```

---

## 外观模式（Facade）

这个设计模式由 PHP 编码实现。

#### 外观模式

外部与一个子系统的通信必须通过一个统一的外观对象进行，为子系统中的一组接口提供一个一致的界面，外观模式定义了一个高层接口，这个接口使得这一子系统更加容易使用。外观模式又称为门面模式，它是一种对象结构型模式。

对于新的业务需求，不要修改原有外观类，而应该增加一个新的具体外观类，由新的具体外观类来关联新的子系统对象，同时通过修改配置文件来达到不修改源代码并更换外观类的目的。 

根据 “单一职责原则”，在软件中将一个系统划分为若干个子系统有利于降低整个系统的复杂性，一个常见的设计目标是使子系统间的通信和相互依赖关系达到最小，而达到该目标的途径之一就是引入一个外观对象，它为子系统的访问提供了一个简单而单一的入口。

外观模式也是 “迪米特法则” 的体现，通过引入一个新的外观类可以降低原有系统的复杂度，同时降低客户类与子系统类的耦合度。

外观模式要求一个子系统的外部与其内部的通信通过一个统一的外观对象进行，外观类将客户端与子系统的内部复杂性分隔开，使得客户端只需要与外观对象打交道，而不需要与子系统内部的很多对象打交道。

外观模式的目的在于降低系统的复杂程度。

外观模式从很大程度上提高了客户端使用的便捷性，使得客户端无须关心子系统的工作细节，通过外观角色即可调用相关功能。

在外观模式中，通常只需要一个外观类，并且此外观类只有一个实例，换言之它是一个单例类。在很多情况下为了节约系统资源，一般将外观类设计为单例类。当然这并不意味着在整个系统里只能有一个外观类，在一个系统中可以设计多个外观类，每个外观类都负责和一些特定的子系统交互，向用户提供相应的业务功能。

不要通过继承一个外观类在子系统中加入新的行为，这种做法是错误的。外观模式的用意是为子系统提供一个集中化和简化的沟通渠道，而不是向子系统加入新的行为，新的行为的增加应该通过修改原有子系统类或增加新的子系统类来实现，不能通过外观类来实现。

外观模式最大的缺点在于违背了 “开闭原则”，当增加新的子系统或者移除子系统时需要修改外观类，可以通过引入抽象外观类在一定程度上解决该问题，客户端针对抽象外观类进行编程。对于新的业务需求，不修改原有外观类，而对应增加一个新的具体外观类，由新的具体外观类来关联新的子系统对象，同时通过修改配置文件来达到不修改源代码并更换外观类的目的。

#### Laravel 外观模式

Laravel 中像 `LogServiceFacade::saveLog();` 这样使用的外观模式需要 IOC 容器提供支持。

下面在 **KongZhiFanZhuan.md** 那篇的代码的基础上，给出日志服务外观模式的代码。

```php
// 外观模式需要 IOC 容器支持
require_once 'KongZhiFanZhuan.php';

class LogServiceFacade
{
    // 维护 Ioc 容器
    protected static $ioc; 

    public static function setFacadeIoc($ioc)
    {
        static::$ioc = $ioc;
    }

    // 返回 LogService 在 Ioc 中的 bind 的 key
    protected static function getFacadeAccessor()
    {
        return 'LogService';
    }

    // php 魔术方法，在静态上下文中调用一个不可访问方法时会被调用
    public static function __callStatic($method, $args)
    {
        $instance = static::$ioc->make(static::getFacadeAccessor());
        // ... 是可变数量的参数列表的语法，表示把 $args（通常是数组）依次赋值给 method() 的参数表
        return $instance->$method(...$args);
    }
}
```

测试代码如下：

```php
$ioc = new IOC();
$ioc->bind('LogInterface', 'DataBaseLog');
$ioc->bind('LogService', 'LogService');

LogServiceFacade::setFacadeIoc($ioc);
LogServiceFacade::saveLog();
```

#### Laravel 外观模式工作原理

Laravel Facade 的核心实现原理就是在 LogServiceFacade 中提前注入 IOC 容器。

定义一个服务提供者的外观类，在该类定义一个类的变量，变量值和在 IOC 容器绑定抽象类时使用的 key 一样。这个操作是为了让注入外观类的 IOC 容器构造实例。

使用静态方法的方式调用目标方法时，通过静态魔术方法 __callStatic() 在调用目标方法之前调用 IOC 容器的 make() 方法构造出实例，然后再调用实例的目标方法。

---

## 参考

[设计模式--外观模式](https://learnku.com/docs/laravel-kernel/design-mode-appearance-mode/6917)

[Facades外观模式背后实现原理](https://learnku.com/docs/laravel-core-concept/5.5/Facades/3020)