## 异常案例

两个Nginx+PHP+MySQL的环境交互。由于网络请求受阻，一段时间后，环境B访问环境A时产生如下报错。

```
upstream timed out (110: Connection timed out) while reading response header from upstream
```

推测原因是，环境A由于访问环境B的CURL超时时间设置为30S，所以网络受阻后，大量的请求在CURL阶段进入等待状态，但是这些请求依然在占用PHP-FPM的worker进程，刚开始等待中的请求数量还在环境A的可承受范围内，所以没有上述报错。

一段时间后阻塞的CURL耗尽了环境A的PHP-FPM的worker进程，导致后续环境B访问环境A时，Nginx分配不出worker进程，导致如上报错。