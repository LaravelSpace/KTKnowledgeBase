## 在Homestead中安装Swoole扩展

这里推荐使用**pecl**安装，执行`$ pecl install swoole`命令安装Swoole扩展。这里有可能会出现下面两种问题。

第一种问题：

```
vagrant@homestead:~/code$ pecl install swoole
WARNING: channel "pecl.php.net" has updated its protocols, use "pecl channel-update pecl.php.net" to update
Cannot install, php_dir for channel "pecl.php.net" is not writeable by the current user
```

这种按照提示执行`$ sudo pecl channel-update pecl.php.net`就可以。

```
vagrant@homestead:~$ sudo pecl channel-update pecl.php.net
Updating channel "pecl.php.net"
Update of Channel "pecl.php.net" succeeded
```

第二种问题：

```
vagrant@homestead:~$ pecl install swoole
Cannot install, php_dir for channel "pecl.php.net" is not writeable by the current user
```

需要在前面加`$ sudo`

```
vagrant@homestead:~$ sudo pecl install swoole
downloading swoole-4.5.1.tgz ...
Starting to download swoole-4.5.1.tgz (1,491,368 bytes)
......................................................................................................................................................................................................................................................................................................done: 1,491,368 bytes
344 source files, building
running: phpize
Configuring for:
PHP Api Version:         20170718
Zend Module Api No:      20170718
Zend Extension Api No:   320170718
```

安装Swoole时的部分配置，这里按需配置就行。

```
enable sockets supports? [no] : yes
enable openssl support? [no] : no
enable http2 support? [no] : no
enable mysqlnd support? [no] : yes
```

看见这个就表示安装成功了。

```
Build process completed successfully
Installing '/usr/lib/php/20170718/swoole.so'
Installing '/usr/include/php/20170718/ext/swoole/config.h'
install ok: channel://pecl.php.net/swoole-4.5.1
configuration option "php_ini" is not set to php.ini location
You should add "extension=swoole.so" to php.ini
```

最后要在**正在使用**的PHP版本的php.ini中添加swoole扩展。可以通过`$ php -i | grep php.ini`命令，查看php.ini文件的位置。

```
vagrant@homestead:/etc/php/7.2/fpm$ php -i | grep php.ini
Configuration File (php.ini) Path => /etc/php/7.2/cli
Loaded Configuration File => /etc/php/7.2/cli/php.ini
```

这里需要注意使用`$ sudo vim php.ini`打开并编辑文件，默认没有文件的写权限，直接打开会报下面的错。

```
[ErrorException]
file_put_contents(./composer.json): failed to open stream: Permission denied
```

保存后，使用`$ php -m`命令查看扩展是否生效。

```
vagrant@homestead:/etc/php/7.2/cli$ php -m | grep swoole
swoole
```

---

| 参考来源                                                     |
| ------------------------------------------------------------ |
| [centos7.2安装swoole扩展](https://www.jianshu.com/p/fa2cbf1a9e26) |

