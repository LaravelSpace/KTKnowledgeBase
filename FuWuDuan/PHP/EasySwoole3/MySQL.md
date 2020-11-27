## 使用MySQL

### 客户端模式

客户端模式没啥好说的。

### 连接池模式

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

MySQL的配置需要在配置文件里加一下。

```php
'mysql_db' => [
    'host'     => '127.0.0.1',
    'port'     => '3306',
    'user'     => 'homestead',
    'password' => 'secret',
    'database' => 'homestead',
    'charset'  => 'utf8mb4'
]
```

然后基于通用连接池实现一个MySQL连接池。

```php
use EasySwoole\Pool\AbstractPool;

class MysqliPool extends AbstractPool
{
    public function __construct(\EasySwoole\Pool\Config $config)
    {
        parent::__construct($config);
    }

    protected function createObject()
    {
        // 从配置文件获取配置
        $configData = \EasySwoole\EasySwoole\Config::getInstance()->getConf('mysql_db');
        // 创建mysqli配置类
        $mysqliConfig = new \EasySwoole\Mysqli\Config($configData);
        // 创建一个mysqli客户端
        return new MysqliClient($mysqliConfig);
    }
}
```

连接池创建对象这里，先从配置文件获取MySQL的配置，然后用这些配置创建一个mysqli客户端。如果有连接不同MySQL数据库的需求，可以考虑MySQL的配置参数也从外部传入。

到这里自定义的连接池就完成了，在使用之前还需要注册连接池，由于需要在所有的调用点之前注册，所以比较合适的地方是EasySwooleEvent.php文件的initialize()方法。

```php
// 设置连接池参数
$mysqliPoolConfig = new \EasySwoole\Pool\Config();
$mysqliPoolConfig->setMinObjectNum(3);
$mysqliPoolConfig->setMaxObjectNum(5);
// 注册mysql连接池
\EasySwoole\Pool\Manager::getInstance()->register(new MysqliPool($mysqliPoolConfig), 'mysql-pool');
```

使用的话就很简单了。需要注意的就是，用完之后记得归还mysqli客户端对象。

```php
// 获取连接池对象
$mysqlPool = \EasySwoole\Pool\Manager::getInstance()->get('mysql-pool');
// 获取mysqli客户端
$mysqliClient = $mysqlPool->getObj();
// sql操作
$mysqliClient->queryBuilder()->get('airbnb_listing');
$dbData = $mysqliClient->execBuilder();
// 释放mysqli客户端
$mysqlPool->recycleObj($mysqliClient);
```

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

### ORM模式

ORM模式的使用相对简单一点，不需要想自定义MySQL连接池那样额外创建什么东西。首先还是安装依赖。

```shell
composer require easyswoole/pool
composer require easyswoole/mysqli
composer require easyswoole/orm
```

然后就是在EasySwooleEvent.php文件的initialize()方法里注册相关信息了。

```php
// 从配置文件获取配置
$configData = \EasySwoole\EasySwoole\Config::getInstance()->getConf('mysql_db');
// 设置orm配置参数
$ormConfig = new \EasySwoole\ORM\Db\Config($configData);
$ormConfig->setMinObjectNum(5);
$ormConfig->setMaxObjectNum(10);
// 注册orm配置参数
\EasySwoole\ORM\DbManager::getInstance()->addConnection(new \EasySwoole\ORM\Db\Connection($ormConfig));
```

这样ORM的连接池就配置好了，ORM会自行调用，然后是具体的ORM使用流程了，另外ORM模式框架自身提供了事务操作。如果有需要访问不同数据库的需求，在构造连接池的时候，设定一个名字就可以了，然后再ORM模型中配置使用的连接池的名字。默认情况下这两个配置的名字都叫default。

