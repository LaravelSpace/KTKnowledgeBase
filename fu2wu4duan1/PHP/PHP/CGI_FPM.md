## CGI，FastCGI，PHP-CGI，PHP-FPM的关系

### 问题背景

想要了解CGI，FastCGI，PHP-CGI，PHP-FPM的关系，首先需要了解一些Web服务的基础知识。在Web服务中，WebServer一般指Apache、Nginx、Tomcat、IIS、Lighttpd等服务器，WebApplication一般指PHP、Java、.NET等应用程序。在整个网站架构中，WebServer（Apache,Nginx）只是内容的分发者。

举个例子，如果客户端请求的是index.html，那么WebServer会去文件系统中找到这个文件，发送给浏览器，这里分发的是静态数据。

![](E:\GongZuoQu\ZhiShiKu\TuPian\FuWuDuan\PHP\CGI_FPM02.png)

如果请求的是index.php，根据配置文件，WebServer知道这个不是静态文件，需要去找PHP解析器来处理，那么就会把这个请求简单处理，然后交给PHP解析器。

![](E:\GongZuoQu\ZhiShiKu\TuPian\FuWuDuan\PHP\CGI_FPM04.png)

当WebServer收到index.php这个请求后，会启动对应的CGI程序，这里就是PHP的解析器。接下来PHP解析器会解析php.ini文件，初始化执行环境，然后处理请求，再以规定CGI规定的格式返回处理后的结果，退出进程，WebServer再把结果返回给浏览器。这就是一个完整的动态PHPWeb访问流程。

### CGI,FastCGI,PHP-CGI,PHP-FPM

- CGI：全称通用网关接口（CommomGatewayInterface）。是WebServer与WebApplication之间数据交换的一种协议。
- FastCGI：是CGI的升级版本，FastCGI可以看成是一个常驻型的CGI。同CGI，是一种通信协议，但比CGI在效率上做了一些优化。同样，SCGI与FastCGI类似。
- PHP-CGI：是PHP（WebApplication）对WebServer提供的CGI协议的接口程序。
- PHP-FPM：全称PHP FastCGI进程管理器（FastCGI Process Manager）。是PHP（WebApplication）对WebServer提供的FastCGI协议的接口程序，额外还提供了相对智能一些任务管理。
- CLI：别和上面的CGI混淆了，CLI是PHP的命令运行模式。

### Module方式

在了解CGI之前，先了解一下WebServer传递数据的另外一种方法：PHP Module加载方式。以Apache为例，使用PHP Module方式时，在Apache的配置文件httpd.conf中，会作出如下修改：

```ini
#修改
<IfModule dir_module>
    DirectoryIndex index.html index.php
</IfModule>
#添加
<IfDefine php7.2>
    PHPIniDir "E:\PHP\php-7.2.31"
    LoadModule php7_module "E:\PHP\php-7.2.31\php7apache2_4.dll"
    AddType application/x-httpd-php .php .html .htm
</IfDefine>
```

这么做的目的是，把PHP作为Apache的一个子模块来运行。当通过Web访问PHP文件时，Apache就会调用php7_module来解析PHP代码。php7_module通过SAPI来将数据传给PHP解析器来解析PHP代码。

SAPI，即是Server Application Programming Interface的首字母缩写，意思是服务器端应用编程接口。这是PHP内核提供给外部调用其服务的接口，即外部系统可以通过SAPI来调用PHP提供的编译脚本、执行脚本的服务。PHP默认提供了很多种SAPI，常见的提供给Apache和Nginx的php7_module、CGI、FastCGI，给IIS的ISAPI，以及Shell的CLI。下面详细描述Apache、SAPI、PHP的关系。

![](E:\GongZuoQu\ZhiShiKu\TuPian\FuWuDuan\PHP\CGI_FPM06.png)

从上图中，可以看出SAPI处于中间位置，SAPI提供了一个和外部通信的接口，使得PHP可以和其他应用（Apache，Nginx等）进行数据交互。在Apache中调用PHP脚本执行的过程为：Apache=>httpd=>php7_module=>SAPI=>PHP。

这种将PHP模块安装到Apache中的模式，在每一次Apache接收请求时，都会产生一个进程来连接PHP通过SAPI来完成请求，这个进程会完整的包括PHP的各种运算计算等操作。这带来一个问题，一旦并发数过多，服务器就会承受不住。

### CGI

CGI：全称通用网关接口（Commom Gateway Interface），是Web服务器与PHP应用进行“交谈”的一种工具，是HTTP服务器和动态脚本语言间通信的接口或者工具，其程序需要运行在网络服务器上。CGI可以用任何一种语言编写，只要这种语言具有标准输入、输出和环境变量，如php、perl、tcl等。CGI就是规定要传哪些数据，以及以什么样的格式传递给后方处理这个请求的协议。它规定了Web服务器会传哪些数据，如URL、查询字符串、POST数据、HTTP HEADER等，给PHP解析器。

CGI就是专门用来和Web服务器打交道的。Web服务器收到用户请求，就会把请求提交给CGI程序（如PHP-CGI），CGI程序根据请求提交的参数作应处理（解析PHP），然后输出标准的html语句，返回给Web服服务器，Web服务器再返回给客户端，这就是通常的CGI的工作原理。

CGI的好处是完全独立于任何服务器，仅做为中间结构，把动态语言解析和HTTP服务器分离开来。提供接口给Apache和PHP。他们通过CGI来完成数据传递。这样做的好处是减少了二者的关联，使得二者可以更加独立。但是CGI也有一个缺陷，就是每一次Web请求都有启动和退出过程，也就是**fork-and-execute**模式，一旦并发数过多，就会出问题。

### FastCGI

FastCGI：是CGI的升级版本，本质上和CGI是一个差不多的东西。是用来提高CGI程序性能的。CGI解释器的反复加载是CGI性能低下的主要原因，如果CGI解释器保持在内存中，并接受FastCGI进程管理器调度，则可以提供良好的性能。FastCGI接口方式采用C/S架构，分为客户端（HTTP服务器）和服务端（动态语言解析服务器）。

FastCGI可以看成是一个常驻（long-live）型的CGI。激活后，可以一直执行着，不会每次都要花时间去启动一次。FastCGI程序也可以和Web服务器分别部署在不同的主机上，它还可以接受来自其他Web服务器的请求，它可以支持分布式。FastCGI是语言无关的。其主要行为是将CGI解释器进程保持在内存中并因此获得高效的性能。

FastCGI进程管理器需要单独启动。启动FastCGI后，会生成一个FastCGI主进程和多个子进程（子进程其实就是CGI解释器进程）。当客户端请求Web服务器上的动态脚本时，Web服务器会将动态脚本通过TCP协议交给FastCGI主进程，FastCGI主进程根据情况，安排一个空闲的子进程来解析动态脚本，处理完成后将结果返回给Web服务器，Web服务器再将结果返回给客户端。该客户端请求处理完毕后，FastCGI子进程并不会随之关闭，而是继续等待主进程安排工作任务。

### FastCGI的工作原理

![](E:\GongZuoQu\ZhiShiKu\TuPian\FuWuDuan\PHP\CGI_FPM08.png)

1、WebServer启动时载入FastCGI进程管理器（Apache Module或IIS ISAPI等）。

2、FastCGI进程管理器先进行自身的初始化，解析配置文件，初始化执行环境，启动一个FastCGI主进程。然后启动多个CGI解释器子进程（PHP-CGI），并等待来自WebServer的连接。与CGI模式的区别是，FastCGI不需要每个请求都重新解析php.ini、重新载入全部扩展，并重初始化全部数据结构。在FastCGI中，这个操作只会在启动PHP-CGI时执行一次，然后PHP-CGI便会保存在内存中不会退出。但是由于FastCGI是多进程的，所以比CGI多线程消耗更多的服务器内存。

3、当客户端请求到达WebServer时，FastCGI进程管理器选择并连接到一个CGI解释器（即分配一个PHP-CGI），并将CGI环境变量和标准输入发送到FastCGI子进程PHP-CGI然后立即可以接受下一个请求。这样就避免了只使用CGI时重复的启动和退出的问题。而且当子进程PHP-CGI不够用时，FastCGI主进程可以根据配置预先启动几个PHP-CGI等着。同理，空闲的子进程PHP-CGI太多时，也会停掉一些，这样就提高了性能，也节约了资源。

4、子进程PHP-CGI完成处理后，将标准输出和错误信息从同一连接返回WebServer。当FastCGI子进程关闭连接时，请求便处理完成。FastCGI子进程进入等待状态，等待处理来自FastCGI进程管理器的下一个连接。在CGI模式中，PHP-CGI在此时会退出。

### PHP-FPM

PHP-FPM：全称PHP FastCGI进程管理器（FastCGI Process Manager）。PHP-FPM就是PHP中的FastCGI进程管理器。PHP-FPM的管理对象是PHP-CGI。PHP-FPM提供了更好的PHP进程管理方式，可以有效控制内存和进程、可以平滑重载PHP配置。


