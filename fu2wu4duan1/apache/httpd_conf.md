## 配置Apache的httpd.conf文件

### 在Windows10操作系统下的Apache服务的配置

首先解压下载下来的Apache压缩包，然后放到合适的位置。我这里用的是**Apache2.4**版本的压缩包**httpd-2.4.43-lre302-x64-vc14.rar**。

Apache的配置文件是httpd.conf。它在根目录的conf目录下，可以以文档的方式打开该文件。

#### 配置Apache的根目录

```
Define SRVROOT "/Apache24"
ServerRoot "${SRVROOT}"
```

**Define SRVROOT**参数定义了一个变量，这和各个编程语言里定义的变量用途是一样的，便于重复利用。这个参数定义了你的Apache根目录。

```
Define SRVROOT "E:\Apache\Apache24"
ServerRoot "${SRVROOT}"
```

#### 配置Apache的监听端口

```
#Listen 12.34.56.78:80
Listen 80
```

这里默认是上面这样的，表示Apache运行的时候监听的是localhost的80端口。如果想要配置成Laravel默认启动的8000端口或者Java常用的8080端口直接改成对应的端口号就可以了。我这里配置的是8010端口。

```
#Listen 12.34.56.78:80
Listen 8010
```

#### 配置默认页

默认页的用途是：在访问Apache的根目录时，如果发现目录下有这些文件，就会直接展示这些文件的内容。

```
#
# DirectoryIndex: sets the file that Apache will serve if a directory
# is requested.
#
<IfModule dir_module>
    DirectoryIndex index.html
</IfModule>
```

为了让Apache把index.php也设为默认页，需要把index.php加到后面，像下面这样。

```
<IfModule dir_module>
    DirectoryIndex index.html index.php
</IfModule>
```

#### 配置网络目录

```
DocumentRoot "${SRVROOT}/htdocs"
<Directory "${SRVROOT}/htdocs">
    #
    # Possible values for the Options directive are "None", "All",
    # or any combination of:
    #   Indexes Includes FollowSymLinks SymLinksifOwnerMatch ExecCGI MultiViews
    #
    # Note that "MultiViews" must be named *explicitly* --- "Options All"
    # doesn't give it to you.
    #
    # The Options directive is both complicated and important.  Please see
    # http://httpd.apache.org/docs/2.4/mod/core.html#options
    # for more information.
    #
    Options Indexes FollowSymLinks

    #
    # AllowOverride controls what directives may be placed in .htaccess files.
    # It can be "All", "None", or any combination of the keywords:
    #   AllowOverride FileInfo AuthConfig Limit
    #
    AllowOverride FileInfo

    #
    # Controls who can get stuff from this server.
    #
    Require all granted
</Directory>
```

默认情况下网络目录会在Apache根目录htdocs目录下，但是有时需要修改这个目录的位置。这个时候就需要修改**DocumentRoot**这个参数，把它修改成目标目录。通常情况下和**DocumentRoot**和**Directory**这两个参数都指向同一个目录，要不然会报错。

```
DocumentRoot "E:\WorkspacePHP"
<Directory "E:\WorkspacePHP">
    Options Indexes FollowSymLinks
    AllowOverride FileInfo
    Require all granted
</Directory>
```

我这里修改成我自己的目录。这样在访问localhost:8010时，Apache就会访问E:\WorkspacePHP这个目录下的内容，我可以在目录下放一个index.php。

#### 添加PHP模块

最后在配置文件最后添加PHP模块，让Apache可以解析PHP代码。

```
#指定PHP的配置文件php.ini所在的目录
PHPIniDir "{path to php}\php-7.1.23"
#指定Apache加载PHP模块
LoadModule php7_module "{path to php}\php-7.1.23\php7apache2_4.dll"
AddType application/x-httpd-php .php .html .htm
```

#### 安装Apache服务

当配置都完成之后，以管理员身份运行Win10的**cmd.exe**。进入Apache根目录的bin目录，执行`httpd –k install`安装Apache服务。这时会把Apache服务注册为Win10的系统服务。默认的服务名是Apache2.4，你可以从**我的电脑=>右击=>服务和应用程序=>服务**里面找到这个Apache服务。

##### 问题1

缺少vcruntime140.dll。

##### 问题1的解决方案

安装Microsoft Visual C++ 2015运行库。

##### 问题2

(OS 10048)通常每个套接字地址(协议/网络地址/端口)只允许使用一次
make_sock: could not bind to address 0.0.0.0:80

##### 问题2的解决方案

打开cmd.exe通过`$ netstat -ano`命令查看端口占用情况，找到调用相关端口的进程和进程PID。然后打开任务管理器，在**查看**菜单中选择**选择列**选项并勾选PID，点击确定。之后就可以通过PID查看占用端口的程序，协调或关闭占用端口的程序就可以解决这个问题。

#### 报错日志

特别的，如果发现问题，可以先去Apache根目录的logs目录，里面的**access.log**和**error.log**会记录Apache服务器上执行的操作和产生的报错。同时也要注意日志的清理或者及时的分片处理，不要让日志太大，不然就很难打开了。

#### 测试Apache

完成后在浏览器访问http://localhost:8010会出现：**It Works**字样或者出现默认配置页index.html和index.php的页面。