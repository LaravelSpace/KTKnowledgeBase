```json
{
  "title": "控制反转（IOC）",
  "updated_at": "2020-05-21",
  "updated_by": "KelipuTe",
  "tags": "设计模式,控制反转,IOC"
}
```

---

## 控制反转（IOC）

这个设计模式由 PHP 编码实现。

IOC（Inversion of Control，缩写为 IOC）翻译过来就是控制反转。在介绍 IOC 容器之前，首先要介绍依赖注入，反射和控制反转的概念。

#### 控制反转（IOC）

首先看这样一段代码：

```php
/**
 * Interface LogInterface 日志接口
 */
interface LogInterface
{
    public function saveLog();
}

/**
 * Class FileLog 文件记录日志
 */
class FileLog implements LogInterface
{
    public function saveLog()
    {
        echo "save log by file" . PHP_EOL;
    }
}

/**
 * Class DatabaseLog 数据库记录日志
 */
class DatabaseLog implements LogInterface
{
    public function saveLog()
    {
        echo "save log by database" . PHP_EOL;
    }
}

// 日志操作类
class LogService
{
    protected $log;

    public function __construct()
    {
        $this->log = new FileLog();
    }

    public function saveLog()
    {
        $this->log->saveLog();
    }
}

$logService = new LogService();
$logService->saveLog();
```

这样的写法其实存在一个问题，如果需要修改 FileLog 变成 DatabaseLog，这时就需要修改 LogService 类。这种写法没达到解耦合（耦合，是对模块间关联程度的度量），也不符合编程开放封闭原则（开放封闭原则的核心的思想是软件实体是可扩展，而不可修改的。也就是说，对扩展是开放的，而对修改是封闭的）。所以需要对 LogService 类进行改进。

```php
/**
 * Class LogService 日志操作类
 */
class LogService
{
    protected $log;

    public function __construct(LogInterface $log)
    {
        $this->log = $log;
    }

    public function saveLog()
    {
        $this->log->saveLog();
    }
}

$logService = new LogService(new DatabaseLog());
$logService->saveLog();
```

这样修改以后只需要通过构造函数参数传递不同的日志接口实例，就可以实现不同的日志记录方式。这种由外部负责解决类的内部的实例创建的行为，可以称其为控制反转。

#### 依赖注入（DI）

上面的例子也算是依赖注入，LogService 类不是由自己内部创建实例，而是通过构造函数传递进来的。通过构造函数或者方法传递实例到类的内部的行为都属于依赖注入。

#### 反射

在面向对象的编码中，对象被赋予了自省的能力，而这个自省的过程就是反射。直观理解就是，根据一个具体的对象，可以获得这个对象的是哪个类，构造函数是什么样的，有哪些属性和方法。

PHP IOC 容器常用的方法有下面这几个。

```php
// 获取 reflectionClass 对象
$reflector = new reflectionClass(LogService::class);
// 拿到构造函数
$constructor = $reflector->getConstructor();
// 拿到构造函数的所有依赖参数
$dependencies = $constructor->getParameters();
// 创建对象，不需要传递参数
$logService = $reflector->newInstance();
// 创建对象，需要传递参数
$logService = $reflector->newInstanceArgs($dependencies = []);
```

使用上面的这几个方法就可以实现简单的 IOC 容器。

#### IOC 容器

IOC 容器的代码如下。

```php
/**
 * Class IOC 容器
 */
class IOC
{
    protected $binding = [];

    /**
     * 绑定抽象类和实例类的关系
     * @param $abstract
     * @param $concrete
     */
    public function bind($abstract, $concrete)
    {
        // 因为 bind() 的时候还不需要创建对象，所以采用 closure，等到 make() 的时候再创建对象
        $this->binding[$abstract]['concrete'] = function ($ioc) use ($concrete) {
            return $ioc->build($concrete);
        };
    }

    /**
     * 创建对象
     * @param $concrete
     * @return object [实例对象]
     * @throws ReflectionException
     */
    public function build($concrete)
    {
        // 获取 reflectionClass 对象
        $reflector = new ReflectionClass($concrete);
        // 拿到构造函数
        $constructor = $reflector->getConstructor();
        if (is_null($constructor)) {
            // 创建对象
            return $reflector->newInstance();
        } else {
            // 拿到构造函数的所有依赖参数
            $dependencies = $constructor->getParameters();
            $instances = $this->getDependencies($dependencies);
            // 创建对象，需要传递参数
            return $reflector->newInstanceArgs($instances);
        }
    }

    /**
     * 获取参数依赖
     * @param $paramters [反射得到的依赖参数]
     * @return array
     */
    protected function getDependencies($paramters)
    {
        $dependencies = [];
        foreach ($paramters as $paramter) {
            $dependencies[] = $this->make($paramter->getClass()->name);
        }
        return $dependencies;
    }

    /**
     * 创建对象
     * @param $abstract
     * @return mixed
     */
    public function make($abstract)
    {
        $concrete = $this->binding[$abstract]['concrete']; // 根据 key 获取 binding 的值

        return $concrete($this);
    }
}
```

实际使用时，就可以为 LogInterface 接口绑定不同的日志实例类。相应的，构造 LogService 类时，具体的日志实例依赖就会由 IOC 容器注入。

```php
$ioc = new IOC();
// $ioc->bind('LogInterface','DataBaseLog');
$ioc->bind('LogInterface','FileLog');
$ioc->bind('LogService','LogService');

$logService = $ioc->make('LogService');
$logService->saveLog();
```

---

## 参考

[依赖注入,控制翻转,反射各个概念的理解和使用](https://learnku.com/docs/laravel-core-concept/5.5/%E4%BE%9D%E8%B5%96%E6%B3%A8%E5%85%A5,%E6%8E%A7%E5%88%B6%E7%BF%BB%E8%BD%AC,%E5%8F%8D%E5%B0%84/3017)

[如何实现Ioc容器和服务提供者是什么概念](https://learnku.com/docs/laravel-core-concept/5.5/Ioc%E5%AE%B9%E5%99%A8,%E6%9C%8D%E5%8A%A1%E6%8F%90%E4%BE%9B%E8%80%85/3019)

[PHP中的反射](https://zhuanlan.zhihu.com/p/99480095)