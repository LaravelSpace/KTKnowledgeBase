## Composer

#### 切换镜像

全局切换镜像。（这里用的是阿里云的镜像）

```
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/
```

全局取消镜像配置。

```
composer config -g --unset repos.packagist
```

只配置当前项目。

```
composer config repo.packagist composer https://mirrors.aliyun.com/composer/
```

取消当前项目镜像配置。

```
composer config --unset repos.packagist
```

#### 更新 composer

使用 composer 时，会遇到提示 composer 版本超过 60 天，建议更新的提示。

这个时候可以使用 `composer self-update` 命令更新 composer。

##### 报错处理

```
> composer selfupdate
Updating to version beb64914a37205f6748552d764d9531d99c7f41e (snapshot channel).
   Downloading (100%)
Failed to decode response: zlib_decode(): data error
Retrying with degraded mode, check https://getcomposer.org/doc/articles/troubleshooting.md#degraded-mode for more info
Downloading (100%)

  [ErrorException]
  zlib_decode(): data error

self-update [-r|--rollback] [--clean-backups] [--no-progress] [--update-keys] [--stable] [--preview] [--snapshot] [--set-channel-only] [--] [<version>]
```

这种格式的报错，比较玄学的解决方法是：把全局设置的镜像路由里 **https** 改成 **http**。



