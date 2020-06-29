```json
{
  "title": "配置 Apache vhost",
  "updated_at": "2020-06-29",
  "updated_by": "KelipuTe",
  "tags": "Apache,Laravel"
}
```

## 配置 Apache vhost

####  Laravel 配置 Apache vhost

首先文档内容提示：

```
Laravel 中包含了一个 public/.htaccess 文件，通常用于在资源路径中隐藏 index.php 的前端控制器。在用 Apache 为 Laravel 提供服务之前，确保启用了 mod_rewrite 模块，这样 .htaccess 文件才能被服务器解析。
```

这需要在 Apache 配置文件 httpd.conf 中找到 `LoadModule rewrite_module modules/mod_rewrite.so` 去掉前面的 # 符号，启动 mod_rewrite 模块。

要是要用 vhost 还需要找到 `Include conf/extra/httpd-vhosts.conf` 去掉前面的 # 符号，启动 vhost 配置文件。vhost 文件的位置在 Apache 根目录 conf/extra/httpd-vhosts.conf。

在 httpd-vhosts.conf 添加项目的配置，如下：

```
<VirtualHost *:80>
    ServerAdmin xhy_1365@sina.com
    DocumentRoot "E:\WorkspacePHP\KTCommunity\public"
    ServerName localhost
    ErrorLog "logs/ktcommunity-error.log"
    CustomLog "logs/ktcommunity-access.log" common
</VirtualHost>
```

##### 报错1

访问 localhost 看见下面这样的提示：

```
Internal Server Error

The server encountered an internal error or misconfiguration and was unable to complete your request.

Please contact the server administrator at  xhy_1365@sina.com to inform them of the time this error occurred, and the actions you performed just before this error.

More information about this error may be available in the server error log.
```

查看报错文件 logs/ktcommunity-error.log，显示如下报错：

```
[core:alert] [pid 7224:tid 2204] [client 127.0.0.1:57026] E:/WorkspacePHP/KTCommunity/public/.htaccess: Options not allowed here
```

##### 报错1的解决方法

去掉 public/.htaccess 中的部分代码。

```
<IfModule mod_negotiation.c>
        Options -MultiViews
</IfModule> 
```

网上搜到的解释是这样的：

```
A3.1的apache是2.4的，规则里不支持Options -MultiViews中的 - 号，这里你可以直接把以下代码去掉
<IfModule mod_negotiation.c>
        Options -MultiViews
</IfModule>
因为这个和apache2.4的规则冲突，并且环境默认配置过，没必要重复申明。 
```

这么干确实有用，但是问什么有用，没有深究。

## 参考

[Laravel 7 中文文档--入门指南--安装](https://learnku.com/docs/laravel/7.x/installation/7447)

[部署Laravel 失败 .htaccess: Options not allowed here](https://php.upupw.net/apache/6/517.html)