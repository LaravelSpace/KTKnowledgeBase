## 使用Redis

### 连接池模式

在EasySwoole中有两种方式实现redis连接池，一种是使用通用连接池组件自定义实现，另一种是直接安装Redis-Pool组件。

#### 使用Redis-Pool组件

首先需要安装依赖，Redis-Pool是基于pool通用连接池和redis协程客户端封装的组件。

```shell
composer require easyswoole/pool
composer require easyswoole/redis
composer require easyswoole/redis-pool
```

安装完成后先在配置文件中添加redis的相关配置。

```php
'redis_db' => [
    'host'           => '127.0.0.1', // ip
    'port'           => '6379', // 端口
    'auth'           => 'password', // 密码
    'db'             => 0, // 数据库
    'timeout'        => 3.0, // 超时时间
    'reconnectTimes' => 3，// 客户端异常重连次数
    'serialize'      => 0, // 数据是否序列化
]
```

在使用之前还需要注册连接池，由于需要在所有的调用点之前注册，所以比较合适的地方是EasySwooleEvent.php文件的initialize()方法。

```php
// 从配置文件获取配置
$configData = \EasySwoole\EasySwoole\Config::getInstance()->getConf('redis_db');
// 创建redis配置
$redisConfig = new \EasySwoole\Redis\Config\RedisConfig($configData);
// 注册redis连接池
$redisPool = \EasySwoole\RedisPool\Redis::getInstance()->register('redis-pool', $redisConfig);
// 设置连接池参数
$redisPool->setMinObjectNum(3);
$redisPool->setMaxObjectNum(5);
```

具体使用时需要注意的是，在用完之后一定要归还（释放）redis连接对象，否则连接池资源耗尽后，后续程序就无法继续获得redis连接对象了。

```php
// 获取连接池对象
$redisPool = \EasySwoole\RedisPool\Redis::getInstance()->get('redis-pool');
// 获取redis连接
$redisObj = $redisPool->getObj();
// sql操作
$redisObj->set('test-redis-pool','test redis pool data');
$dbData = $redisObj->get('test-redis-pool');
// 释放redis连接
$redisPool->recycleObj($redisObj);
```

### 客户端模式

。。。