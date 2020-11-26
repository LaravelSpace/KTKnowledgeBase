## 在Apache中配置多个版本的PHP

### 在Windows10环境的Apache中配置两个及以上版本的PHP

这里用**php-5.6.40**和**php-7.1.23**举例配置两个版本的PHP的过程。

按照常规的配置方法分别配置好就行，不需要添加系统变量。

#### 配置Apache

这里使用Apache2.4举例。在配置单个版本的PHP的时候，会在httpd.conf文件里添加下面这段代码。其中`{path to php}`表示各个版本PHP的目录。

```
#指定PHP的配置文件php.ini所在的目录
PHPIniDir "{path to php}\php-7.1.23"
#指定Apache加载PHP模块
LoadModule php7_module "{path to php}\php-7.1.23\php7apache2_4.dll"
AddType application/x-httpd-php .php .html .htm
```

在命令行中执行`$ httpd.exe -k install`可以注册Apache服务。如果是Apache2.4的版本，默认注册的服务名就是Apache2.4 。这里也可以自定义服务名 `$ httpd.exe -k install -n Apache2.4_php7.1`。

在命令行中执行`$ sc delete Apache2.4`可以卸载服务名为Apache2.4的服务。

#### 注册两个不同版本PHP的Apache服务

首先修改httpd.conf文件，把上面的那段代码改成下面这样。

```
<IfDefine php5.6>
    PHPIniDir "{path to php}\php-5.6.40"
    LoadModule php5_module "{path to php}\php-5.6.40\php5apache2_4.dll"
    AddType application/x-httpd-php .php .html .htm
</IfDefine>

<IfDefine php7.1>
    PHPIniDir "{path to php}\php-7.1.23"
    LoadModule php7_module "{path to php}\php-7.1.23\php7apache2_4.dll"
    AddType application/x-httpd-php .php .html .htm
</IfDefine>
```

然后在命令行分别执行下面两个命令，注册两个Apache2.4服务。`-n`表示自定义命名，`-D`参数用于上面的`<IfDefine xxx>`。

```shell
$ httpd.exe -k install -n Apache2.4_php5.6 -D php5.6
$ httpd.exe -k install -n Apache2.4_php7.1 -D php7.1
```

完成之后就可以使用**ApacheMonitor**启动不同的Apache服务，服务会对应使用各自配置的PHP版本。启动好服务后可以输出`phpinfo()`检查一下。

#### 卸载Apache

首先关闭Apache服务（方式很多，就不一一列举了）。然后在命令行中运行命令：`$ sc delete Apache2.4_php5.6`删除Apache服务。

---

| 参考来源                                                     |
| ------------------------------------------------------------ |
| [apache 配置多个版本的 php](https://www.cnblogs.com/songlen/p/6613884.html) |
| [Apache服务器下载、安装、启动、关闭及卸载（win版）](https://blog.csdn.net/wd2011063437/article/details/79088346) |