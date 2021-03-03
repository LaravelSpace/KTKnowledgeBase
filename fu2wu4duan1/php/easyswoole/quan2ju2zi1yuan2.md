## 全局资源

在EasySwoole框架可以通过全局变量，全局常量还有单例对象使用一些全局资源。

### 全局变量

在Swoole协程当中`$_GET`，`$_POST`等全局变量是不能安全使用的。原因是协程切换下会带来数据污染问题。Easyswoole在spl包中，实现了一个SplContextArray，并在主进程的位置，替换了这些全局变量，这使得这些数据的访问是安全的，并在请求结束后自动清理。

可以通过GlobalParamHook实现`$_GET`，`$_POST`等全局变量的安全使用。注意：该特性需要1.6版本以上的http组件库`"easyswoole/http": "^1.6"`。在EasySwooleEvent.php文件中，通过如下方式注册和初始化全局变量。这样注册完成之后就可以在任意地方使用这些全局变量了。

```php
public static function mainServerCreate(EventRegister $register)
{
    GlobalParamHook::getInstance()->hookDefault();
}

public static function onRequest(Request $request, Response $response): bool
{
    GlobalParamHook::getInstance()->onRequest($request, $response);
}
```

这里需要注意，上面面说的任意地方不包括协程内部。如果碰到需要在协程中使用`$_GET`，`$_POST`中的数据时，一定要在进入协程之前把数据取出来，然后传到协程里去。在协程中这些全局变量中的数据并不存在。

### 全局常量

全局常量有多种实现方式。

##### 配置文件

EasySwoole框架提供了基础的dev.php和produce.php两个配置文件，根据框架启动命令的不同，只会加载其中一个，所以如果是线上环境和开发环境一样的配置就会要复制一遍，不是太方便的。而且这两个配置文件在框架进入EasySwooleEvent.php文件的initialize()方法之前就加载了，而不修改源码的情况下，initialize()方法是开发者可以进行操作的第一个方法。如果想要修改配置文件的加载逻辑，是无法在不修改框架源码的情况下修改的。

另外一点，像账号或者密码这样的敏感数据也不太适合直接放在代码里提交到代码库里去。所以合理的解决方案是额外引入一个配置文件放置敏感信息或者使用数据库加缓存的模式，对于不是特别敏感的配置项就放到常量里去维护。

框架说明文档里特别提到dev.php和produce.php这两个配置文件是不能热重启的。只有onWorkerStart事件之后加载的文件，才能被`$ php easyswoole reload`命令热重启。

##### .env文件

由于框架提供的配置文件不是那么的好用，所以使用.env文件也是个不错的选择。首先使用composer命令安装依赖：`$ composer require vlucas/phpdotenv`，然后创建.env文件编写配置参数，然后把.env文件加载到程序里去就行了。之后就可以通过$_ENV全局变量使用这些参数了。

```php
$dotenv = \Dotenv\Dotenv::createImmutable(EASYSWOOLE_ROOT, '.env');
$dotenv->load();
```

值得注意的一点是，配置文件中如果有数据库密码这种全局通用信息的，一定要在所有的业务代码用到配置信息之前，把配置信息加载到程序里去。所以比较合理的位置是EasySwooleEvent.php文件的initialize()方法。

##### 静态常量

静态常量是最简单的全局常量是实现方式。直接在业务代码里使用静态常量定义出来就行了，这种实现方式可以满足热重启的需要，比较适合保存一些不敏感的全局配置信息。

##### 数据库+缓存

数据库加缓存的模式也可以解决保存配置信息的问题。但是，和数据库以及缓存有关的配置信息是不能通过这个方法保存的。由于配置信息数据通常是持续有效的所以可以把缓存的失效时间去掉，另外可以加一些接口便于从外部操作数据库和缓存中保存的数据。

### 单例对象

单例对象主要就是指EasySwoole框架提供的各种单例模式的服务了，在这些单例对象中可以适当地保存一些相关的全局资源。