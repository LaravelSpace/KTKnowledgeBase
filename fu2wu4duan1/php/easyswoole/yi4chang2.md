## 异常

### 启动时异常

在执行启动服务时输出以下异常信息：

```
[2020-11-10 17:17:52][debug][error]:[EasySwoole\Task\Worker bind /home/vagrant/code/Temp/EasySwoole.TaskWorker.0.sock fail case Operation not permitted at file:/home/vagrant/code/vendor/easyswoole/component/src/Process/Socket/AbstractUnixProcess.php line:31]
[2020-11-10 17:17:52][debug][error]:[EasySwoole\Task\Worker bind /home/vagrant/code/Temp/EasySwoole.TaskWorker.1.sock fail case Operation not permitted at file:/home/vagrant/code/vendor/easyswoole/component/src/Process/Socket/AbstractUnixProcess.php line:31]
[2020-11-10 17:17:52][debug][error]:[EasySwoole\Task\Worker bind /home/vagrant/code/Temp/EasySwoole.TaskWorker.2.sock fail case Operation not permitted at file:/home/vagrant/code/vendor/easyswoole/component/src/Process/Socket/AbstractUnixProcess.php line:31]
[2020-11-10 17:17:52][debug][error]:[EasySwoole\Task\Worker bind /home/vagrant/code/Temp/EasySwoole.TaskWorker.3.sock fail case Operation not permitted at file:/home/vagrant/code/vendor/easyswoole/component/src/Process/Socket/AbstractUnixProcess.php line:31]
[2020-11-10 17:17:52][debug][error]:[EasySwoole\EasySwoole\Bridge\BridgeProcess bind /home/vagrant/code/Temp/bridge.sock fail case Operation not permitted at file:/home/vagrant/code/vendor/easyswoole/component/src/Process/Socket/AbstractUnixProcess.php line:31]
```

根据报错信息可以判断是创建socket的时候系统报错：`Operation not permitted`，原因是缺少`pid.pid`文件，这个文件应该是自动创建的，这里的原因是虚拟机共享目录不支持创建文件的操作，所以这个文件没有被创建出来。解决方案是到easyswoole的配置文件中将TEMP_DIR值修改为linux系统的临时目录：`/tmp`。