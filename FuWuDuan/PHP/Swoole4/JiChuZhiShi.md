## 基础知识

### Swoole几个进程和线程之间的关系

#### Master进程

Master进程是一个多线程进程。主进程内有多个Reactor线程，基于epoll/kqueue进行网络事件轮询。收到数据后转发到Worker进程去处理。

#### Reactor线程

Reactor线程是在Master进程中创建的线程。负责维护客户端TCP连接、处理网络IO、处理协议、收发数据。将TCP客户端发来的数据缓冲、拼接、拆分成完整的一个请求数据包。不执行任何PHP代码。

#### Worker进程

Worker以多进程的方式运行，可以是异步非阻塞模式，也可以是同步阻塞模式。对收到的数据进行处理，包括协议解析和响应请求。接受由Reactor线程投递的请求数据包，并执行PHP回调函数处理数据。生成响应数据并发给Reactor线程，由Reactor线程发送给TCP客户端。如果未设置Server->worker_num，底层会启动与CPU数量一致的Worker进程。

当Worker进程内发生致命错误或者人工执行exit()时，进程会自动退出。Master进程会重新启动一个新的Worker进程来继续处理请求。

#### TaskWorker进程

TaskWorker以多进程的方式运行，完全是同步阻塞模式。接受由Worker进程通过Swoole\Server->task/taskwait/taskCo/taskWaitMulti()方法投递的任务。处理任务，并将结果数据返回（使用Swoole\Server->finish()）给Worker进程。

#### Manager进程

负责创建/回收Worker/Task进程。对所有Worker进程进行管理，Worker进程生命周期结束或者发生异常时自动回收，并创建新的Worker进程。

它们之间的关系可以理解为Reactor就是nginx，Worker就是PHP-FPM。Reactor线程异步并行地处理网络请求，然后再转发给Worker进程中去处理。Reactor和Worker间通过unixSocket进行通信。

在PHP-FPM的应用中，经常会将一个任务异步投递到Redis等队列中，并在后台启动一些PHP进程异步地处理这些任务。Swoole的Reactor、Worker、TaskWorker之间可以紧密的结合起来，提供更高级的使用方式。Swoole提供的TaskWorker是一套更完整的方案，将任务的投递、队列、PHP任务处理进程管理合为一体。通过底层提供的API可以非常简单地实现异步任务的处理。另外TaskWorker还可以在任务执行完成后，再返回一个结果反馈到Worker。

