## 在Lunem中使用Redis

### 安装Redis扩展

注释格式设置在代码格式设置的地方，菜单栏=>File=>Settings=>Editor=>Code Style=>PHP。

```
$ composer require predis/predis
$ composer require illuminate/redis
```

安装时需要注意版本。直接执行上面的命令安装illuminate/redis时会安装最新版本的扩展，如果Lumen是较低的版本，有可能会安装不成功。这个时候就需要指定illuminate/redis的版本。例如：为`Lumen 5.6.*`安装illuminate/redis扩展时，可以使用`$ composer require illuminate/redis=5.6.*`命令指定版本。

2、在bootstrap/app.php文件中注册Redis服务。

```php
// 如果被注释了，需要打开注释
$app->withFacades();
// 如果被注释了，需要打开注释
$app->withEloquent();
// 注册 Redis 服务
$app->register(Illuminate\Redis\RedisServiceProvider::class);
```

3、在`.env`文件中配置相关环境变量。

```
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_DATABASE=0
REDIS_PASSWORD=root
```

这些参数会在config/database.php文件里配置Redis的地方被用到。

### 使用Redis

这个可以参考：[Laravel 大将之 Redis 模块](https://segmentfault.com/a/1190000009695841)。

---

| 参考来源                                                     |
| ------------------------------------------------------------ |
| [Laravel 中文文档](https://learnku.com/docs/laravel/5.8/redis/3930#predis) |
| [Lumen安装使用Redis](https://blog.csdn.net/qq_38191191/article/details/81354599) |
| [Laravel 5.4 使用 Predis 报密码错误的问题](https://www.jianshu.com/p/af238b0fa845) |
| [Composer - illuminate/redis installation fails because of different versions of illuminate/support](https://stackoverflow.com/questions/34443492/composer-illuminate-redis-installation-fails-because-of-different-versions-of) |
| [Laravel 大将之 Redis 模块](https://segmentfault.com/a/1190000009695841) |