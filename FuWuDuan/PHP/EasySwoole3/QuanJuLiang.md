## 全局量的使用

在EasySwoole框架里有两种方式全局使用一些资源，一种是全局变量，另一种是全局常量。

### 全局变量

在Swoole协程当中$\_GET，$\_POST等全局变量是不能安全使用的。原因是协程切换下会带来数据污染问题。Easyswoole在spl包中，实现了一个SplContextArray，并在主进程的位置，替换了这些全局变量，这使得这些数据的访问是安全的，并在请求结束后自动清理。

可以通过GlobalParamHook实现$\_GET，$\_POST等全局变量的安全使用。注意：该特性需要1.6版本以上的http组件库`"easyswoole/http": "^1.6"`。在EasySwooleEvent.php文件中，通过如下方式注册和初始化全局变量。这样注册完成之后就可以在任意地方使用这些全局变量了。

```php
public static function mainServerCreate(EventRegister $register)
{
    // 实现安全使用$_GET,$_POST等全局变量
    GlobalParamHook::getInstance()->hookDefault();
}

public static function onRequest(Request $request, Response $response): bool
{
    // 实现安全使用$_GET,$_POST等全局变量
    GlobalParamHook::getInstance()->onRequest($request, $response);
}
```

### 全局常量

全局常量有两种实现方式，静态常量和配置文件，静态常量没什么好说的，这里主要说一下配置文件。EasySwoole自带的配饰文件有一个不好的地方就是他在EasySwoole.php的initialize()方法之前就使用了，而不修改源码的情况下，initialize()方法是开发者可以进行操作的第一个方法。而且如果把一些敏感的信息直接配置在那两个配饰文件里会存在安全隐患。所以在框架自带配置文件的基础上，又加入了.env配置文件。

首先安装依赖`"vlucas/phpdotenv": "^5.2"`。在EasySwoole.php的initialize()方法的最前面引入.env文件后，就可以在程序中使用$_ENV来使用.env文件中的配置数据。

```php
public static function initialize()
{
    // 加载.env文件
    $dotenv = \Dotenv\Dotenv::createImmutable(EASYSWOOLE_ROOT, '.env');
    $dotenv->load();
}
```



