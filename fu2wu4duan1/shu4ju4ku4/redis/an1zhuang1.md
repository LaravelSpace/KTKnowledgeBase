```json
{
  "title": "在 Windows 中安装 Redis",
  "updated_at": "2020-06-29",
  "updated_by": "KelipuTe",
  "tags": "Windows,Redis"
}
```

---

## 在 Windows 中安装 Redis

#### 下载 Redis

首先去官网下载安装版或者解压版的 `Redis`，这里我去 `GitHub` 下载。
值得注意的是 `GitHub` 上有两个 `Redis` ：一个是 [antirez/redis](https://github.com/antirez/redis) 这个不支持 `Windows`，支持 `Windows` 的是另一个 [MicrosoftArchive/redis](https://github.com/MicrosoftArchive/redis) 。

在使用 `Redis` 的过程中可能还需要一个类似 `Navicat` 的数据库桌面管理系统，这里推荐 `RedisDesktopManager` ，可以去 `Github` 下载 [uglide/RedisDesktopManager](https://github.com/uglide/RedisDesktopManager) 。新版本的 `RedisDesktopManager` 是需要付费的，这里可以先下载不用付费的 [0.8.8](https://github.com/uglide/RedisDesktopManager/releases/tag/0.8.8) 版本的。有能力可以支持一下，费用并不高。

#### 安装 Redis

这里我是用的是解压版的。解压完成后，首先去 `Redis` 文件夹根目录找到 `redis.windows.conf` 文件。这是 `Windows` 系统下 `Redis` 的配置文件。

用记事本打开，一般不需要修改什么配置，我这里修改一下密码。全局搜索 `requirepass`，找到以后去掉前面的 `#` 号，注意有空格的话也去掉，修改后面的密码，这样 `root` 就是 `Redis` 的密码了，保存后关闭。

```
requirepass root
```

其实这样就可以启动 `Redis` 数据库了，但是这里为了后续方便，我需要把 `Redis` 注册为 `Windows` 的服务。从命令行进入 `Redis` 文件夹根目录，常用的操作有下面 4 个。

```
$ redis-server --service-install redis.windows.conf // 安装 redis 服务
$ redis-server --service-start // 启动 redis 服务 
$ redis-server --service-stop // 停止 redis 服务
$ redis-server --service-uninstall // 卸载 redis 服务
```

执行第一行命令后 `Redis` 服务就成为 `Windows` 的服务了，可以从 `右击我的电脑->管理-服务和应用程序->服务` 中找到。名字一般就是 `Redis`。

安装好后需要手动执行第二行命令启动服务。也可以从刚才那个服务管理界面启动。

#### 使用 RedisDesktopManager

这个用过 `Navicat` 的话应该会很熟悉。端口默认就是 6379。

![WindowsAnZhuangRedis_01](./WindowsAnZhuangRedis_01.png)

#### 异常处理 1067 程序意外终止

1、记录日志

找到 **redis.windows.conf** 文件中的 **logfile** 参数。例如，这么设置：`logfile "logs/redis_log.txt"`，日志会被记录在 `{{dir to redis}}/logs/redis_log.txt`。

2、重新安装一次 Redis 服务，并启动。日志文件里获得如下内容：

```
[8344] 22 Apr 11:15:18.166 # Granting read/write access to 'NT AUTHORITY\NetworkService' on: "C:\Study\Database\redis-x64-3.2.100" "C:\Study\Database\redis-x64-3.2.100\" 
[8344] 22 Apr 11:15:18.167 # Redis successfully installed as a service.
[7800] 22 Apr 11:15:23.512 # Creating Server TCP listening socket 127.0.0.1:6379: unable to bind socket
```

3、 检查一下 Windows 服务，看一下这里是不是网络服务。

![WindowsAnZhuangRedis_02](./WindowsAnZhuangRedis_02.png)

如果是网络服务，右击属性，改成本地服务。

![WindowsAnZhuangRedis_03](./WindowsAnZhuangRedis_03.png)

## 参考

[windows下redis安装](https://blog.csdn.net/u012343297/article/details/78839063)

[Redis在windows下安装与配置](https://www.cnblogs.com/lezhifang/p/7027903.html)

[Windows Redis默认配置文件，Redis配置不生效解决方案](https://www.cnblogs.com/fanshuyao/p/7193299.html)

[Windows 下 Redis 服务无法启动，错误 1067 进程意外终止解决方案](https://blog.csdn.net/maxsky/article/details/79775118)