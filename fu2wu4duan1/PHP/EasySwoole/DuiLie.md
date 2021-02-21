## 队列

Easyswoole封装实现了一个轻量级的队列，默认以Redis作为队列驱动器。首先添加依赖：

```shell
composer require easyswoole/queue
```

### 使用流程

使用流程主要有几个步骤：创建队列任务，注册队列驱动器，设置消费进程，生产者投递任务。

#### 创建队列任务

创建队列任务比较简单，就是一个单例。

```php
use EasySwoole\Component\Singleton;
use EasySwoole\Queue\Queue;

class TestQueue extends Queue
{
    use Singleton;
}
```

#### 注册队列驱动器

框架提供的基础的驱动器EasySwoole\Queue\Driver\Redis，使用redis的lpush命令插入任务，使用rpop消费任务。这样的暴力轮询的获取任务的方式对redis数据库的压力比较大，所以建议自己重写一个驱动器使用redis的brpop命令获取任务。brpop命令是阻塞式获取，如果获取不到数据会阻塞给定的等待时间然后返回null，如果在等待时间中有新的数据插入就会获取到数据。

这里给出一个简单实现的使用brpop命令的驱动器，注意brpop命令返回等待超时是预料之内的，需要主动捕获并处理掉。

```php
use EasySwoole\Queue\Job;
use EasySwoole\Queue\QueueDriverInterface;
use EasySwoole\Redis\Exception\RedisException;
use EasySwoole\Redis\Redis as Connection;
use EasySwoole\RedisPool\RedisPool;

class QueueRedisDriver implements QueueDriverInterface
{
    protected $pool;
    protected $queueName;

    public function __construct(RedisPool $pool, string $queueName = 'easy_queue')
    {
        $this->pool = $pool;
        $this->queueName = $queueName;
    }

    public function push(Job $job): bool
    {
        $data = serialize($job);
        return $this->pool->invoke(function (Connection $connection) use ($data) {
            return $connection->lPush($this->queueName, $data);
        });
    }

    public function pop(float $timeout = 3.0): ?Job
    {
        return $this->pool->invoke(function (Connection $connection) use ($timeout) {
            $timeout = intval($timeout);
            try {
                $data = $connection->bRPop($this->queueName, $timeout);
                if (!empty($data) && isset($data[$this->queueName])) {
                    return unserialize($data[$this->queueName]);
                }
            } catch (RedisException $exc) {
                // 判断是不是超时了，brpop超时是预期之内的错误
                $pos = strpos($exc->getMessage(), 'Connection timed out');
                if ($pos === false) {
                    // 如果不是超时，就继续抛异常报错
                    throw $exc;
                }
            }
            return null;
        });
    }

    public function size(): ?int
    {
        return $this->pool->invoke(function (Connection $connection) {
            return $connection->lLen($this->queueName);
        });
    }
}
```

构造完驱动器后，需要注册一个redis连接池给队列驱动器使用，然后设置队列任务使用哪个驱动器。这里需要注意如果两个队列使用了同一个驱动器，那么实际在redis中，它们的队列数据就在同一个list里面。所以这个时候A队列能监听并获取到B队列的任务数据，稍有不慎容易出错，所以建议驱动器分开用。

```php
$redisPool = \EasySwoole\RedisPool\Redis::getInstance()->get('driver-pool');
$driver1 = new QueueRedisDriver($redisPool, 'queue_name1');
Test::getInstance($driver1);
```

#### 设置消费进程

队列定义好之后，需要定义一个消费进程，也就是定义一个自定义进程，用来监听和消费队列。队列模型的listen()方法有4个参数，第一个参数是获取到任务时的回调函数，用于处理任务，第二个参数是每隔多长时间监听一次队列，第三个参数是监听的最长等待时间，第四个参数是最多可以同时运行的任务数量。

```php
use App\Helper\TimeHelper;
use App\Queue\ZhiLian\Ctrip\CalendarQueue;
use App\Service\ZhiLian\Ctrip\CtripService;
use EasySwoole\Component\Process\AbstractProcess;
use EasySwoole\EasySwoole\Logger;
use EasySwoole\Queue\Job;

class TestProcess extends AbstractProcess
{
    protected function run($arg)
    {
        $pid = getmypid();
        Logger::getInstance()->info(timeNow() . ":TestProcess run,pid={$pid}");
        go(function () {
            TestQueue::getInstance()->consumer()->listen(function (Job $job) {
                var_dump($job->getJobData());
            }, 0.5, 3, 5);
        });
    }

    protected function onException(\Throwable $throwable, ...$args)
    {
        parent::onException($throwable, $args);
    }

    protected function onShutDown()
    {
        $pid = getmypid();
        Logger::getInstance()->info(timeNow() . ":TestProcess onShutDown,pid={$pid}");
        parent::onShutDown();
    }
}
```

最后别忘了，需要在EasySwooleEvent.php的mainServerCreate()方法里注册自定义进程。

```php
public static function mainServerCreate(EventRegister $register)
{
    \EasySwoole\Component\Process\Manager::getInstance()->addProcess(new TestProcess());
}
```

