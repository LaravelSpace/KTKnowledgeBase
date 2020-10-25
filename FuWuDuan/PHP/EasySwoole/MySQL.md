## 使用MySQL

### 客户端模式

客户端模式没啥好说的。

### 连接池模式

EasySwoole没有提供现成的mysql连接池，所以这里需要自行实现一个。需要用到pool通用连接池和框架提供的mysqli客户端组件。

首先添加依赖。

```shell
composer require easyswoole/pool
composer require easyswoole/mysqli
```

首先需要实现一个自定义的客户端，用于连接池创建客户端连接对象。这里的配置类从外部传入，方便后续连接不同的MySQL数据库的需求。

```php
use EasySwoole\Mysqli\Client;
use EasySwoole\Pool\ObjectInterface;

class MysqliObject extends Client implements ObjectInterface
{
    /**
     * MysqliObject constructor.
     * @param \EasySwoole\Mysqli\Config $config
     */
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
        // TODO: Implement objectRestore() method.
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
    /**
     * MysqliPool constructor.
     * @param \EasySwoole\Pool\Config $config
     * @throws \EasySwoole\Pool\Exception\Exception
     */
    public function __construct(\EasySwoole\Pool\Config $config)
    {
        parent::__construct($config);
    }

    /**
     * @return MysqliObject
     */
    protected function createObject()
    {
        // 从配置文件获取配置
        $configData = \EasySwoole\EasySwoole\Config::getInstance()->getConf('mysql_db');
        // 创建mysqli配置类
        $mysqliConfig = new \EasySwoole\Mysqli\Config($configData);
        // 创建一个mysqli客户端
        return new MysqliObject($mysqliConfig);
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

使用的话就很简单了。需要注意的就是记得归还mysqli客户端对象。

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

### ORM模式

ORM模式的使用相对简单一点，不需要想自定义MySQL连接池那样额外创建什么东西。

首先还是安装依赖。

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

然后是具体的ORM使用流程了。