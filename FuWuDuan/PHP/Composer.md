## Composer使用方式

### 切换镜像


```shell
#全局切换镜像（这里用的是阿里云的镜像）
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/
#全局取消镜像配置
composer config -g --unset repos.packagist
#只配置当前项目镜像
composer config repo.packagist composer https://mirrors.aliyun.com/composer/
#取消当前项目镜像配置
composer config --unset repos.packagist
```

比较推荐使用只配置当前项目镜像，因为这个配置可以被带到composer.json里去。即使换了环境，再次执行composer命令时也会使用镜像。

### 安装指定版本的扩展

```shell
#安装5.5版本的xxx扩展
composer require xxx ^5.5
#安装dev-master版本的xxx扩展
composer require xxx:dev-master
```

### 删除扩展

```shell
#移除一个
composer remove xxx
#移除多个
composer remove xxx1 xxx2
```

移除扩展只是从composer.json文件中移除了扩展的依赖。扩展移除之后，扩展的代码还是在vender目录里的，只是不会被composer自动加载了。

### 更新composer

使用composer时，会遇到提示composer版本超过60天，建议更新的提示。这个时候可以使用`composer self-update`命令更新composer。

##### 报错1

```shell
$ composer self-update
Updating to version beb64914a37205f6748552d764d9531d99c7f41e (snapshot channel).
   Downloading (100%)
Failed to decode response: zlib_decode(): data error
Retrying with degraded mode, check https://getcomposer.org/doc/articles/troubleshooting.md#degraded-mode for more info
Downloading (100%)

  [ErrorException]
  zlib_decode(): data error

self-update [-r|--rollback] [--clean-backups] [--no-progress] [--update-keys] [--stable] [--preview] [--snapshot] [--set-channel-only] [--] [<version>]
```

##### 报错1的解决方案

这种格式的报错，比较玄学的解决方法是把全局设置的镜像路由里**https**改成**http**。

### 安装ip2region包时报错

```shell
$ composer require zoujingli/ip2region
Using version ^1.0 for zoujingli/ip2region
./composer.json has been updated
Loading composer repositories with package information
Updating dependencies (including require-dev)
PHP Fatal error:  Allowed memory size of 1610612736 bytes exhausted (tried to allocate 4096 bytes) in phar://C:/ProgramData/ComposerSetup/bin/composer.phar/src/Composer/DependencyResolver/RuleSetGenerator.php on line 129

Fatal error: Allowed memory size of 1610612736 bytes exhausted (tried to allocate 4096 bytes) in phar://C:/ProgramData/ComposerSetup/bin/composer.phar/src/Composer/DependencyResolver/RuleSetGenerator.php on line 129

Check https://getcomposer.org/doc/articles/troubleshooting.md#memory-limit-errors for more info on how to handle out of memory errors.
E:\GongZuoQu\innpms-server>composer require zoujingli/ip2region
Using version ^1.0 for zoujingli/ip2region
./composer.json has been updated
Loading composer repositories with package information
Updating dependencies (including require-dev)
PHP Fatal error:  Allowed memory size of 1610612736 bytes exhausted (tried to allocate 4096 bytes) in phar://C:/ProgramData/ComposerSetup/bin/composer.phar/src/Composer/DependencyResolver/RuleSetGenerator.php on line 129

Fatal error: Allowed memory size of 1610612736 bytes exhausted (tried to allocate 4096 bytes) in phar://C:/ProgramData/ComposerSetup/bin/composer.phar/src/Composer/DependencyResolver/RuleSetGenerator.php on line 129

Check https://getcomposer.org/doc/articles/troubleshooting.md#memory-limit-errors for more info on how to handle out of memory errors.
```

这只因为安装包的时候PHP内存溢出了，使用`COMPOSER_MEMORY_LIMIT=-1 composer require ***`命令去掉内存限制，或者修改php.ini文件中的内存最大值配置，或者使用`php -d memory_limit=-1 which composer require ***`命令去掉内存限制。

