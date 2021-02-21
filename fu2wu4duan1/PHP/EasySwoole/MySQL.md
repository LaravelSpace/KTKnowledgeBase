## 使用MySQL

### 连接池模式

#### 注册连接池

EasySwoole没有提供现成的mysql连接池，所以这里需要自行实现一个。需要用到pool通用连接池和框架提供的mysqli客户端组件，先添加依赖。

```shell
composer require easyswoole/pool
composer require easyswoole/mysqli
```

首先需要实现一个自定义的客户端，用于连接池创建客户端连接对象。这里的配置类从外部传入，方便后续连接不同的MySQL数据库的需求。

```php
use EasySwoole\Mysqli\Client;
use EasySwoole\Pool\ObjectInterface;

class MysqliClient extends Client implements ObjectInterface
{
    public function __construct(\EasySwoole\Mysqli\Config $config)
    {
        parent::__construct($config);
    }

    function gc()
    {
        $this->close();
    }

    function objectRestore()
    {
    }

    function beforeUse(): ?bool
    {
        return true;
    }
}
```

客户端构造函数接受一个`\EasySwoole\Mysqli\Config`实例。`\EasySwoole\Mysqli\Config`的构造函数接受一个数组作为参数。然后就可以用这些配置实例创建mysqli客户端了。

```php
$configData = [
    'host'     => '127.0.0.1',
    'port'     => '3306',
    'user'     => 'homestead',
    'password' => 'secret',
    'database' => 'homestead',
    'charset'  => 'utf8mb4'
];
$mysqliConfig = new \EasySwoole\Mysqli\Config($configData);
new MysqliClient($mysqliConfig);
```

然后基于通用连接池实现一个MySQL连接池，配置参数从外部传入，便于后续扩展创建连接不同的数据库的连接池使用。连接池的工作就是创建n个客户端并负责维护它们。

```php
use EasySwoole\Pool\AbstractPool;

class MysqliPool extends AbstractPool
{
    protected $configKey;

    protected $configData;

    public function __construct(\EasySwoole\Pool\Config $config, array $configData)
    {
        $this->configData = $configData;
        parent::__construct($config);
    }

    protected function createObject()
    {
        // 创建mysqli配置类
        $mysqliConfig = new \EasySwoole\Mysqli\Config($this->configData);
        // 创建一个mysqli客户端
        return new MysqliClient($mysqliConfig);
    }
}
```

到这里自定义的连接池就完成了，在使用之前还需要注册连接池，由于需要在所有的业务使用之前就注册好，所以比较合适的地方是EasySwooleEvent.php文件的initialize()方法。不同的连接池可以通过名字加以区分，如果没有名字的话会使用默认的名字。

```php
// 设置连接池参数
$mysqliPoolConfig = new \EasySwoole\Pool\Config();
$mysqliPoolConfig->setMinObjectNum(3);
$mysqliPoolConfig->setMaxObjectNum(5);
// 注册mysql连接池
\EasySwoole\Pool\Manager::getInstance()->register(new MysqliPool($mysqliPoolConfig, $configData), $poolName);
```

#### 连接池的使用

使用的话就很简单了。需要注意的就是，用完之后一定要归还mysqli客户端对象。

```php
// 获取连接池对象
$mysqlPool = \EasySwoole\Pool\Manager::getInstance()->get($poolName);
// 获取mysqli客户端
$mysqliClient = $mysqlPool->getObj();
// sql操作
$mysqliClient->queryBuilder()->get('table_name');
$dbData = $mysqliClient->execBuilder();
// 释放mysqli客户端
$mysqlPool->recycleObj($mysqliClient);
```

#### 事务操作

自主实现的MySQL连接池如果想实现事务操作也需要自行编码，需要用到\Swoole\Coroutine\MySQL类中的begin()，commit()，rollback()等事务操作方法。这个类在Client类执行execBuilder()方法的时候（就是构造完语句最终开始执行的时候调用的方法），通过Client类的connect()方法构造。所以如果想主动控制事务，需要把connect()方法的调用提到外层。这个方法是幂等的，所以不会影响内部的逻辑。

```php
$mysqlPool = Manager::getInstance()->get('mysql-pool');
$mysqliClient = $mysqlPool->getObj();
$mysqliClient->connect();
// 开启事务
$mysqliClient->mysqlClient()->begin();
$mysqliClient->queryBuilder()->get('table_name');
$dbResult1 = $mysqliClient->execBuilder();
// 提交事务
$mysqliClient->mysqlClient()->commit();
$mysqlPool->recycleObj($mysqliClient);
```

#### SQL日志

开发环境可以加一个自动输出SQL执行语句的机制，方便排查问题和SQL调优。实现方式是自定义mysqli客户端的execBuilder()方法，简单一点，复制一下加点逻辑就行了。

可以使用`\EasySwoole\EasySwoole\Core::getInstance()->isDev()`方法判断是不是开发环境。然后mysqli客户端提供了lastQueryBuilder参数保存最后一次sql执行的各项参数，比如getLastQuery()方法就可以获取填充数据之后最终执行的SQL语句。

```php
function execBuilder(float $timeout = null)
{
    $this->lastQueryBuilder = $this->queryBuilder;
    if (\EasySwoole\EasySwoole\Core::getInstance()->isDev()) {
        // 开发环境输出sql
        \EasySwoole\EasySwoole\Logger::getInstance()->info($this->lastQueryBuilder->getLastQuery(), 'sql log');
    }
    $start = microtime(true);
    if ($timeout === null) {
        $timeout = $this->config->getTimeout();
    }
    try {
        $this->connect();
        $stmt = $this->mysqlClient()->prepare($this->queryBuilder()->getLastPrepareQuery(), $timeout);
        $ret = null;
        if ($stmt) {
            $ret = $stmt->execute($this->queryBuilder()->getLastBindParams(), $timeout);
        } else {
            $ret = false;
        }
        if ($this->onQuery) {
            call_user_func($this->onQuery, $ret, $this, $start);
        }
        if ($ret === false && $this->mysqlClient()->errno) {
            throw new Exception($this->mysqlClient()->error);
        }
        return $ret;
    } catch (\Throwable $exception) {
        throw $exception;
    } finally {
        $this->reset();
    }
}
```

### ORM模式

ORM模式的使用相对简单一点，不需要像自定义MySQL连接池那样额外创建一些东西。首先还是安装依赖。

```shell
composer require easyswoole/pool
composer require easyswoole/mysqli
composer require easyswoole/orm
```

然后就是在EasySwooleEvent.php文件的initialize()方法里注册相关信息了。MySQL的配置参数和上面客户端的参数格式是一样的。

```php
$configData = [
    'host'     => '127.0.0.1',
    'port'     => '3306',
    'user'     => 'homestead',
    'password' => 'secret',
    'database' => 'homestead',
    'charset'  => 'utf8mb4'
];
// 设置orm配置参数
$ormConfig = new \EasySwoole\ORM\Db\Config($configData);
$ormConfig->setMinObjectNum(3);
$ormConfig->setMaxObjectNum(5);
// 注册orm配置参数
\EasySwoole\ORM\DbManager::getInstance()->addConnection(new \EasySwoole\ORM\Db\Connection($ormConfig), $connectionName);
```

这样ORM的连接池就配置好了，ORM会自行调用，然后是具体的ORM使用流程了，另外ORM模式框架自身提供了事务操作。如果有需要访问不同数据库的需求，在构造连接池的时候，设定一个名字就可以了，然后在ORM模型中配置$connectionName参数指定连接池的名字。默认情况下构造连接池和ORM模型的这两个配置的名字都叫default。

#### ORM的SQL日志

ORM的SQL日志可以通过ORM提供的onQuery事件实现。

```php
\EasySwoole\ORM\DbManager::getInstance()->onQuery(function ($res, $builder, $start) {
    $builder->getLastQuery()
});
```

这里的$builder参数就是上文MySQL连接池中提到的lastQueryBuilder参数，通过$builder的getLastQuery()方法，就可以拿到填充数据之后最终执行的SQL语句。

### 客户端模式

在上文的连接池模式中，定义了一个自定义的mysqli客户端，连接池的工作就是创建了n个mysqli客户端。所以如果想直接使用客户端，那么使用同样的方式，直接创建一个出来就可以了。不过使用完之后记得要使用客户端的gc()方法回收资源。额外需要注意的一点mysqli是框架提供的协程客户端，使用的时候环境必须是协程环境，也就是说用的时候要放到go(function(){})里面去。

