## 命令

### 默认命令

EasySwoole框架提供了一些默认的命令，下面是几个比较常用的命令。

```shell
# 开发模式启动
php easyswoole start
# 守护模式启动
php easyswoole start d
# 开发模式启动,使用produce.php配置文件
php easyswoole start produce
# 守护模式启动,使用produce.php配置文件
php easyswoole start produce d
# 停止服务
php easyswoole stop
# 停止服务,使用produce.php配置文件
php easyswoole stop produce
# 强制停止服务,并重新启动
php easyswoole restart
# 强制停止服务,并重新启动,使用produce.php配置文件
php easyswoole restart produce
# 热重启服务
php easyswoole reload
# 热重启服务,使用produce.php配置文件
php easyswoole reload produce
```

值得注意的有：

- 1、守护模式下才需要stop或者reload，不然control+c或者是终端断开就退出进程了。
- 2、当start命令增加produce之后，其他相关的stop，restart，reload命令都需要增加produce参数，否则可能出错。
- 3、reload命令只会更新onWorkerStart事件之后才加载的文件。而主进程，比如，那两个配置文件和EasySwoole.php文件中的代码，不会被重启。另外，还有http自定义路由的配置也不会被更新。

### 自定义命令

#### 定义自定义命令

除了默认的命令，框架也提供了自定义命令的实现入口。首先实现框架提供命令接口并编写相关的业务逻辑。commandName()方法定义命令的操作名，比如下面的代码定义的一个自定义命令就可以通过`php easyswoole test-command`调用。exec()方法里写业务逻辑。

```php
use EasySwoole\EasySwoole\Command\CommandInterface;

class TestCommand implements CommandInterface
{
    public function commandName(): string
    {
        return 'test-command';
    }

    public function exec(array $args): ?string
    {
        // 业务逻辑
        return null;
    }

    public function help(array $args): ?string
    {
        return 'test command';
    }
}
```

实现完自定义命令，我们还需要把它注册到框架服务里去，这个逻辑在bootstrap.php文件里实现。

```php
\EasySwoole\EasySwoole\Command\CommandContainer::getInstance()->set(new \App\Command\TestCommand());
```

到这里自定义命令就实现完了，已经可以通过`php easyswoole test-command`调用了。

#### 使用自定义命令

通常情况下自定义命令的逻辑按照正常的逻辑写就行了，但是需要注意的是，在主进程中引入的配置文件，注册的连接池，设置的各个单例服务的各项参数在自定义命令里都是不能用的。因为这些操作都是在`php easyswoole start`命令的流程里完成的，自定义命令并不会走这个流程。所以当需要配置文件时，需要在自定义命令的流程中手动引入，当需要连接数据库时，也需要手动创建数据库客户端来满足需求。