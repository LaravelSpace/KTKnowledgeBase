# 在 Lunem 中使用 Redis

## 安装 Redis 扩展

1、使用 `Composer` 安装扩展。

```
$ composer require predis/predis
$ composer require illuminate/redis
```

**注意：安装时需要注意版本。**
直接执行上面的命令安装 `illuminate/redis` 时会安装最新版本的扩展，如果 `Lumen` 是较低的版本，有可能会安装不成功。这个时候就需要指定 `illuminate/redis` 的版本。

例如：为 `Lumen 5.6.*` 安装 `illuminate/redis` 扩展时，可以使用下面的命令指定版本。
```
$ composer require illuminate/redis=5.6.*
```

2、在 `bootstrap/app.php` 中注册 `Redis` 服务。

```php
$app->withFacades(); // 如果被注释了，需要打开注释
$app->withEloquent(); // 如果被注释了，需要打开注释
    
$app->register(Illuminate\Redis\RedisServiceProvider::class); // 注册 Redis 服务
```

3、在 `.env` 文件中配置相关环境变量。

```
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_DATABASE=0
REDIS_PASSWORD=root
```

这些参数会在 `config/database.php` 文件里配置 `Redis` 的地方被用到。

## 使用 Redis

这个可以参考 [Laravel 大将之 Redis 模块](https://segmentfault.com/a/1190000009695841)

## 参考

- [Laravel 中文文档](https://learnku.com/docs/laravel/5.8/redis/3930#predis)
- [Lumen安装使用Redis](https://blog.csdn.net/qq_38191191/article/details/81354599)
- [Composer - illuminate/redis installation fails because of different versions of illuminate/support](https://stackoverflow.com/questions/34443492/composer-illuminate-redis-installation-fails-because-of-different-versions-of)
- [Laravel 5.4 使用 Predis 报密码错误的问题](https://www.jianshu.com/p/af238b0fa845)