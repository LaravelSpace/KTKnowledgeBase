## 在Apache上部署Laravel框架

### 在Windows10操作系统下的Apache服务中配置Laravel框架

首先注意Laravel项目中public目录下的.htaccess文件，文档内容提示：

> Laravel 中包含了一个 public/.htaccess 文件，通常用于在资源路径中隐藏 index.php 的前端控制器。在用 Apache 为 Laravel 提供服务之前，确保启用了 mod_rewrite 模块，这样 .htaccess 文件才能被服务器解析。

#### 直接在httpd.conf文件里配置

和直接配置PHP的环境类似，区别在于配置网络目录，要一直到Laravel项目的public目录才行。这样在运行的时候，Apache服务才能找到位于public目录下的index.php文件。

```
DocumentRoot "E:\WorkspacePHP\KTCommunity\public"
<Directory "E:\WorkspacePHP\KTCommunity\public">
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>
```

保存后启动Apache服务，然后访问配置的路由，如果正常运行就可以看到Laravel项目的欢迎页。

##### 问题1

访问不了路由，提示连接超时或者服务器无响应，或者出现下面这样的提示。

```
Internal Server Error

The server encountered an internal error or misconfiguration and was unable to complete your request.

Please contact the server administrator at  xhy_1365@sina.com to inform them of the time this error occurred, and the actions you performed just before this error.

More information about this error may be available in the server error log.
```

这个时候就需要去查一下Apache的错误日志error.log看看有没有报错。有一种情况会在error.log中找到下面这样的提示。

```
[Tue Jul 28 15:51:43.298840 2020] [core:alert] [pid 6156:tid 2316] [client 127.0.0.1:52443] E:/WorkspacePHP/KTCommunity/public/.htaccess: Options not allowed here
```

##### 问题1的解决方案1

检查一下Apache配置中的AllowOverride选项是否配置为`AllowOverride All`。如果配置的是`AllowOverride None`时是会忽略.htaccess的，这时框架提供的.htaccess文件就无法发挥作用。

##### 问题1的解决方案2

去掉public/.htaccess中的下面这部分代码。这么做可以解决问题，但是不建议这么干，不到万不得已，不要修改官方提供的配置文件。

```
<IfModule mod_negotiation.c>
        Options -MultiViews
</IfModule> 
```

####  通过配置vhost

这需要在Apache配置文件httpd.conf中找到 `LoadModule rewrite_module modules/mod_rewrite.so` 去掉前面的`#`符号，启动mod_rewrite模块。

这里要用vhost，还需要找到 `Include conf/extra/httpd-vhosts.conf` 去掉前面的`#`符号，启动vhost配置文件。vhost文件的位置在Apache根目录conf/extra/httpd-vhosts.conf。

在httpd-vhosts.conf中首先把下面这段注释掉。

```
<VirtualHost _default_:80>
    DocumentRoot "${SRVROOT}/htdocs"
</VirtualHost>
```

再添加我们要配置的项目的配置。

```
<VirtualHost *:80>
    DocumentRoot "E:\WorkspacePHP\KTCommunity\public"
    ServerName localhost.ktcommunity
    ErrorLog "logs/ktcommunity-error.log"
    CustomLog "logs/ktcommunity-access.log" common
</VirtualHost>
```

这里还需要注意的是，要在httpd.conf文件里对上面DocumentRoot参数配置的目录进行配置。

```
<Directory "E:\WorkspacePHP\KTCommunity\public">
    Options Indexes FollowSymLinks
    AllowOverride ALL
    Require all granted
</Directory>
```

配置完成后重启Apache，这个时候我们就可以通过localhost或者127.0.0.1访问了。当然这里也可以通过ServerName参数配置的localhost.ktcommunity这个域名访问，这需要在hosts文件里添加重定向`127.0.0.1 localhost.ktcommunity`。

---

| 参考来源                                                     |
| ------------------------------------------------------------ |
| [Laravel 7 中文文档--入门指南--安装](https://learnku.com/docs/laravel/7.x/installation/7447) |
| [部署Laravel 失败 .htaccess: Options not allowed here](https://php.upupw.net/apache/6/517.html) |

