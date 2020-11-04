## 全局变量

在Swoole协程当中$\_GET，$\_POST等全局变量是不能安全使用的。原因是协程切换下会带来数据污染问题。Easyswoole在spl包中，实现了一个SplContextArray，并在主进程的位置，替换了这些全局变量，这使得这些数据的访问是安全的，并在请求结束后自动清理。

可以通过GlobalParamHook实现$\_GET，$\_POST等全局变量的安全使用。注意：该特性需要1.6版本以上的http组件库`"easyswoole/http": "^1.6"`。

在EasySwooleEvent.php文件中，通过如下方式注册和初始化全局变量。

```php
public static function mainServerCreate(EventRegister $register)
{
    // 实现安全使用$_GET,$_POST等全局变量
    GlobalParamHook::getInstance()->hookDefault();
    ...
}

public static function onRequest(Request $request, Response $response): bool
{
    $fd = $request->getSwooleRequest()->fd;
    $ip = ServerManager::getInstance()->getSwooleServer()->getClientInfo($fd)['remote_ip'];
    echo $ip . PHP_EOL;
    // 实现安全使用$_GET,$_POST等全局变量
    GlobalParamHook::getInstance()->onRequest($request, $response);
    $_GET['ip'] = $ip;
    ...
}
```

