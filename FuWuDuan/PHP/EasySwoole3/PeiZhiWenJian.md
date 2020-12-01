## 配置文件

EasySwoole框架提供了基础的dev.php和produce.php两个配置文件，根据框架启动命令的不同，只会加载其中一个，所以如果是线上环境和开发环境一样的配置就会要复制一遍，不是太方便的。而且这两个配置文件在框架进入EasySwooleEvent.php文件的initialize()方法之前就加载了，如果想要修改配置文件的加载逻辑，是无法在不修改框架源码的情况下修改的。

另外一点，像账号或者密码这样的敏感数据也不太适合直接放在代码里提交到代码库里去。所以合理的解决方案是额外引入一个配置文件放置敏感信息或者使用数据库加缓存的模式，对于不是特别敏感的配置项就放到常量里去维护。

框架说明文档里特别提到dev.php和produce.php这两个配置文件是不能热重启的。只有onWorkerStart之后加载的文件，才能被`$ php easyswoole reload`命令热重启。

### .env文件

首先使用composer命令安装依赖：`$ composer require vlucas/phpdotenv`。然后配置文件如果有数据库密码这种全局通用信息的，一定要在所有的业务代码用到配置信息之前，把配置信息加载到程序里去。所以比较合理的位置是EasySwooleEvent.php文件的initialize()方法。在initialize()方法中使用下面的代码加载.env文件，然后就可以通过$_ENV变量使用这些参数了。

```php
$dotenv = \Dotenv\Dotenv::createImmutable(EASYSWOOLE_ROOT, '.env');
$dotenv->load();
```

