## Swoole\Server

Swoole\Server类是所有异步风格服务器的基类，Http\Server、WebSocket\Server、Redis\Server都继承于它。

### set()方法

```php
Swoole\Server->set(array $setting): void
```

set()方法用于设置运行时的各项参数，服务器启动后通过Server->setting属性来访问设置的参数数组。Server->set()方法必须在Server->start()方法前调用。对于Server的配置即Server->set()方法中传入的参数设置，必须关闭/重启整个Server才可以重新加载。

### $setting属性

Server->set()函数所设置的参数会保存到Server->$setting属性上。在回调函数中可以访问运行参数的值。

##### reactor_num

设置启动的Reactor线程数，默认值为CPU核数。通过此参数来调节主进程内事件处理线程的数量，以充分利用多核。默认会启用CPU核数相同的数量。每个线程能都会维持一个EventLoop。线程之间是无锁的，指令可以被128核CPU并行执行。考虑到操作系统调度存在一定程度的性能损失，可以设置为CPU核数*2，以便最大化利用CPU的每一个核，建议设置为CPU核数的1-4倍。

reactor_num必须小于或等于worker_num。如果设置的reactor_num大于worker_num，会自动调整使reactor_num等于worker_num。在超过8核的机器上reactor_num默认设置为8。

##### worker_num

设置启动的Worker进程数，默认值为CPU核数。开的进程越多，占用的内存就会大大增加，而且进程间切换的开销就会越来越大。所以这里适当即可，不要配置过大。如果业务代码是全异步IO的，这里设置为CPU核数的1-4倍最合理。如果业务代码为同步IO，需要根据请求响应时间和系统负载来调整。默认设置为swoole_cpu_num()，最大不得超过swoole_cpu_num()*1000。

##### max_request

设置worker进程的最大任务数，默认值为0，即不会退出进程。一个Worker进程在处理完超过此数值的任务后将自动退出，进程退出后会释放所有内存和资源。这个参数的主要作用是解决由于程序编码不规范导致的PHP进程内存泄露问题。PHP应用程序有缓慢的内存泄漏，但无法定位到具体原因、无法解决，可以通过设置max_request临时解决，需要找到内存泄漏的代码并修复，而不是通过此方案。达到max_request不一定马上关闭进程，参考max_wait_time。

##### task_worker_num

配置Task进程的数量，没有默认值，未配置则不启动task。配置此参数后将会启用task功能。所以Server务必要注册onTask、onFinish2个事件回调函数。如果没有注册，服务器程序将无法启动。Task进程是同步阻塞的。最大值不得超过swoole_cpu_num()*1000。Task进程内不能使用Swoole\Server->task()方法。

##### daemonize

守护进程化，默认值为0。设置daemonize为1时，程序将转入后台作为守护进程运行。长时间运行的服务器端程序必须启用此项。如果不启用守护进程，当ssh终端退出后，程序将被终止运行。启用守护进程后，CWD（当前目录）环境变量的值会发生变更，相对路径的文件读写会出错，PHP程序中必须使用绝对路径。标准输入和输出会被重定向到log_file。如果未设置log_file，将重定向到/dev/null，所有打印屏幕的信息都会被丢弃。

##### reload_async

设置异步重启开关，默认值为true。设置异步重启开关。设置为true时，将启用异步安全重启特性，Worker进程会等待异步事件完成后再退出。reload_async开启的主要目的是为了保证服务重载时，协程或异步任务能正常结束。

##### max_wait_time

设置Worker进程收到停止服务通知后最大等待时间，默认值为3。经常会碰到由于worker阻塞卡顿导致worker无法正常reload，无法满足一些生产场景，例如发布代码热更新需要reload进程。所以，加入了进程重启超时时间的选项。

管理进程收到重启、关闭信号后或者达到max_request时，管理进程会重起该worker进程。分以下几个步骤：1、底层会增加一个(max_wait_time)秒的定时器，触发定时器后，检查进程是否依然存在，如果是，会强制杀掉，重新拉一个进程。2、需要在onWorkerStop回调里面做收尾工作，需要在max_wait_time秒内做完收尾。3、依次向目标进程发送SIGTERM信号，杀掉进程。

##### log_file

指定Swoole错误日志文件。在Swoole运行期发生的异常信息会记录到这个文件中，默认会打印到屏幕。
开启守护进程模式后(daemonize=>true)，标准输出将会被重定向到log_file。在PHP代码中echo/var_dump/print等打印到屏幕的内容会写入到log_file文件。在服务器程序运行期间日志文件被mv移动或unlink删除后，日志信息将无法正常写入，这时可以向Server发送SIGRTMIN信号实现重新打开日志文件。

##### log_rotation

设置Server日志分割，默认值为SWOOLE_LOG_ROTATION_SINGLE。

| 常量                             | 说明   | 版本信息 |
| -------------------------------- | ------ | -------- |
| SWOOLE_LOG_ROTATION_SINGLE       | 不启用 | -        |
| SWOOLE_LOG_ROTATION_MONTHLY      | 每月   | v4.5.8   |
| SWOOLE_LOG_ROTATION_DAILY        | 每日   | v4.5.2   |
| SWOOLE_LOG_ROTATION_HOURLY       | 每小时 | v4.5.8   |
| SWOOLE_LOG_ROTATION_EVERY_MINUTE | 每分钟 | v4.5.8   |

### start()方法

```php
Swoole\Server->start(): bool
```

start()方法用于启动服务器，监听所有TCP/UDP端口。启动失败会立即返回false。启动成功后会创建Server->worker_num+2个进程。Master进程+Manager进程+Server->worker_num个Worker进程。设置了task_worker_num会增加相应数量的Task进程。启动成功后将进入事件循环，等待客户端连接请求，start()方法之后的代码不会执行。服务器关闭后，start()函数返回true，并继续向下执行。

### reload()方法

```php
Swoole\Server->reload(bool $only_reload_taskworkrer = false): bool
```

reload()方法用于安全地重启所有Worker/Task进程。后端服务器随时都有可能在处理请求，如果通过kill进程方式来终止/重启服务器程序，可能导致刚好代码执行到一半终止。这种情况下会产生数据的不一致。Swoole提供了柔性终止/重启的机制，只需要向Server发送特定的信号，Server的Worker进程就可以安全的结束，并重新拉起。

但有几点要注意：

1、reload有保护机制，当一次reload正在进行时，收到新的重启信号会丢弃。

2、新修改的代码必须要在OnWorkerStart事件中重新载入才会生效，比如某个类在OnWorkerStart之前就通过composer的autoload载入了就是不可以的。Reload操作只能重新载入Worker进程启动后加载的PHP文件，使用get_included_files函数来列出哪些文件是在WorkerStart之前就加载的PHP文件，在此列表中的PHP文件，即使进行了reload操作也无法重新载入。要关闭服务器重新启动才能生效。

3、reload还要配合这两个参数max_wait_time和reload_async，设置了这两个参数之后就能实现异步安全重启。

### stop()方法

```php
Swoole\Server->stop(int $workerId = -1, bool $waitEvent = false): bool
```

stop()方法用于使当前Worker进程停止运行，并立即触发onWorkerStop回调函数。异步IO服务器在调用stop退出进程时，可能仍然有事件在等待。这时如果进程强制退出，这些流程处理的数据就会丢失了。设置`$waitEvent=true`后，底层会使用异步安全重启策略。先通知Manager进程，重新启动一个新的Worker来处理新的请求。当前旧的Worker会等待事件，直到事件循环为空或者超过max_wait_time后，退出进程，最大限度的保证异步事件的安全性。