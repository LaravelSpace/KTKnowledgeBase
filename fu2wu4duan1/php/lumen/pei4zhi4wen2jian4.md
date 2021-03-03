## 在Lumen中使用配置文件

### 在Lumen中使用Laravel的配置方式

想在Lumen中如果想使用Laravel的配置方式，可以把Lumen源代码中Laravel的配置文件复制到config目录下。配置文件可以在`vendor\laravel\lumen-framework\config`目录下找到。例如：如果在Lumen中想同时连接两个数据库。这个时候需要把vendor的config目录下的database.php配置文件，复制到config目录下。使用方式和在Laravel中的使用方式是一样的。

### 在Laravel中同时连接两个数据库

首先配置database.php，增加一个MySQL链接。

```php
'mysql2' => [
    'driver'    => 'mysql',
    'host'      => 'host2',
    'database'  => 'database2',
    'username'  => 'user2',
    'password'  => 'password2',
    'charset'   => 'utf8mb4',
    'collation' => 'utf8mb4_unicode_ci',
    'prefix'    => '',
]
```

配置完成后，就可以在程序中使用mysql2链接另外一个数据库了。下面举了几个例子。

1、在数据库迁移时，可以指定数据表使用的链接。

```php
Schema::connection('mysql2')->create('some_table', function($table) {
    $table->increments('id');
});
```

2、在使用查询构造器时，可以指定数据库连接。

```php
DB::connection('mysql2')->select(...);
```

3、在模型中，可以指定模型数据库连接。

```php
class UserModel extends Eloquent
{
    protected $connection = 'mysql2';
}
```

4、也可以实例化模型之后，再指定数据库连接。

```php
$userModel = new UserModel;
$userModel->setConnection('mysql2');
$user = $userModel->find(1);
// on() 方法也是可以的
$userModel->on('mysql2')->find(1); 
```

### 加载config目录的配置文件

在Laravel中，我们可以使用config()读取配置在config目录下的自定义配置文件。但是在Lumen中，这种使用方式有一个注意点。在新建config目录并创建配置文件如：message.php后，需要在bootstrap/app.php中调用`$app->configure('message');`代码进行手动加载。

原因是：Lumen框架下的服务都是按需加载的。例如：从源码中复制database.php文件到config目录下。如果没有调用过DB(或者打开`$app->withEloquent()`)，直接使用config('database')会返回Null。调用过DB(或者打开`$app->withEloquent()`)后，使用config('database')才会返回配置文件的内容。返回Null是因为不进行上述数据库相关的操作，程序里没有别的地方会加载database.php。

---

| 参考来源                                                     |
| ------------------------------------------------------------ |
| [Lumen 文档](https://learnku.com/docs/lumen/5.7/configuration/2403) |

