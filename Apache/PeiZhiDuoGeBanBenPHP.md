```json
{
  "title": "Apache 服务配置多个版本 PHP",
  "updated_at": "2020-06-29",
  "updated_by": "KelipuTe",
  "tags": "Apache,PHP"
}
```

## Apache 服务配置多个版本 PHP

---

### 在 Windows 10 中，配置两个版本的 PHP

这里用 php-5.6.40 和 php-7.1.23 举例。

按照常规的配置方法分别配置好就行，不需要添加系统变量。

#### 配置 Apache

这里使用 Apache2.4 举例。

在配置单个版本的 php 的时候，会在 httpd.conf 文件里添加下面这段代码。

```
# 指定 PHP 的配置文件 php.ini 所在的目录
PHPIniDir "{path to php}\php-7.1.23"
# 指定 Apache 加载 PHP 模块
LoadModule php7_module "{path to php}\php-7.1.23\php7apache2_4.dll"
AddType application/x-httpd-php .php .html .htm
```

在命令行中执行 `$ httpd.exe -k install` 可以注册 Apache 服务。

如果是 Apache2.4 的版本，默认注册的服务名就是 Apache2.4 。

这里也可以自定义服务名 `$ httpd.exe -k install -n Apache2.4_php7.1`。

在命令行中执行 `$ sc delete Apache2.4` 可以卸载服务名为 Apache2.4 的服务。

#### 注册两个不同版本 PHP 的 Apache 服务器

首先修改 httpd.conf 文件，把上面的那段代码改成下面这样。

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

然后在命令行分别执行下面两个命令，注册两个 Apache2.4 服务。`-n` 表示自定义命名，`-D` 参数用于上面的 `<IfDefine xxx>`。

```
$ httpd.exe -k install -n Apache2.4_php5.6 -D php5.6
$ httpd.exe -k install -n Apache2.4_php7.1 -D php7.1
```

完成之后就可以使用 **ApacheMonitor** 启动不同的 Apache 服务，服务会对应使用各自配置的 PHP 版本。启动好服务后可以输出 `phpinfo()` 检查一下。

#### 卸载 Apache

首先关闭 Apache 服务（方式很多）。然后在命令行中运行命令：`$ sc delete Apache2.4_php5.6`

## 参考

[apache 配置多个版本的 php](https://www.cnblogs.com/songlen/p/6613884.html)

[Apache服务器下载、安装、启动、关闭及卸载（win版）](https://blog.csdn.net/wd2011063437/article/details/79088346)
