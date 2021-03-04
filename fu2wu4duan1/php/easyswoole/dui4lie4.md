## 队列

### 3.x版本

3.x版本的队列组件提供了队列引擎，不过队列引擎的构造方法传入的是redis的配置，由构造方法创建连接池。

```php
public function __construct(RedisConfig $config,string $queueName = 'es_q')
{
    $this->pool = new Pool($config);
    $this->queueName = $queueName;
}
```

这样构造出来的连接池，从外部来看就是个黑盒，如果想要对连接池进行监控的，可以自行重写一个队列引擎，稍微改造一下构造方法，传入构造好的数据库连接池。

```php
public function __construct(Pool $pool, string $queueName = 'easy_queue')
{
    $this->pool = $pool;
    $this->queueName = $queueName;
}
```

