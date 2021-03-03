## 协程高级

### 系统API

Coroutine\System，系统相关API的协程封装。此模块在v4.4.6正式版本后可用。v4.4.6以前的版本，请使用Co短名或Swoole\Coroutine调用，Co::sleep或Swoole\Coroutine::sleep。v4.4.6及以后版本官方推荐使用Co\System::sleep或Swoole\Coroutine\System::sleep。(向下兼容，v4.4.6版本以前的写法也是可以的，无需修改)

##### fread()

```php
Swoole\Coroutine\System::fread(resource $handle, int $length = 0): string|false
```

协程方式读取文件。`resource $handle`是文件句柄，必须是fopen打开的文件类型stream资源。`int $length`设置读取的长度，默认为0，表示读取文件的全部内容。读取成功返回字符串内容，读取失败返回 false。

v4.0.4以下版本fread方法不支持非文件类型的stream，如STDIN、Socket，请勿使用fread操作此类资源。v4.0.4以上版本fread方法支持了非文件类型的stream资源，底层会自动根据stream类型选择使用AIO线程池或EventLoop实现。

##### fwrite()

```php
Swoole\Coroutine\System::fwrite(resource $handle, string $data, int $length = 0): int|false
```

协程方式向文件写入数据。`resource $handle`是文件句柄，必须是fopen打开的文件类型stream资源。`string $data`设置要写入的数据内容，可以是文本或二进制数据。`int $length`设置写入文件的长度，默认为0，表示写入$data的全部内容，$length必须小于$data的长度。写入成功返回数据长度，读取失败返回 false。

v4.0.4以下版本fread方法不支持非文件类型的stream，如STDIN、Socket，请勿使用fread操作此类资源。v4.0.4以上版本fread方法支持了非文件类型的stream资源，底层会自动根据stream类型选择使用AIO线程池或EventLoop实现。

##### fgets()

```php
Swoole\Coroutine\System::fgets(resource $handle): string|false
```

协程方式按行读取文件内容。`resource$handle`是文件句柄，必须是fopen打开的文件类型stream资源。底层使用了php_stream缓存区，默认大小为8192字节，可使用stream_set_chunk_size()设置缓存区尺寸。

读取到EOL（\r或\n）将返回一行数据，包括EOL。未读取到EOL，但内容长度超过php_stream缓存区8192字节，将返回8192字节的数据，不包含EOL。达到文件末尾EOF时，返回空字符串，可用feof判断文件是否已读完。读取失败返回false，使用swoole_last_error()函数获取错误码。

fgets函数仅可用于文件类型的stream资源，Swoole版本>=v4.4.4可用。

##### readFile()

```php
Swoole\Coroutine\System::readFile(string $filename): string|false
```

协程方式读取文件。`string $filename`是文件名。读取成功返回字符串内容，读取失败返回false，可使用swoole_last_error获取错误信息。readFile()方法没有尺寸限制，读取的内容会存放在内存中，因此读取超大文件时可能会占用过多内存。

##### writeFile()

```php
Swoole\Coroutine\System::writeFile(string $filename, string $fileContent, int $flags): bool
```

协程方式写入文件。`string $filename`是文件名，必须有可写权限，文件不存在会自动创建。打开文件失败会立即返回false。`string $fileContent`是写入到文件的内容，最大可写入4M。`int $flags`是写入的选项，默认会清空当前文件内容，可以使用FILE_APPEND表示追加到文件末尾。写入成功返回true，写入失败返回false。

##### sleep()

```php
Swoole\Coroutine\System::sleep(float $seconds): void
```

进入等待状态。`float $seconds`设置睡眠的时间，必须大于0，单位是秒，最小精度为毫秒（0.001秒），最大不得超过一天时间（86400 秒）。相当于PHP的sleep函数，不同的是Coroutine::sleep是协程调度器实现的，底层会yield当前协程，让出时间片，并添加一个异步定时器，当超时时间到达时重新resume当前协程，恢复运行。使用sleep接口可以方便地实现超时等待功能。

### Coroutine\WaitGroup

在Swoole4中可以使用Channel实现协程间的通信、依赖管理、协程同步。基于Channel可以很容易地实现Golang的sync.WaitGroup功能。

```php
Co\run(function () {
    $wg = new \Swoole\Coroutine\WaitGroup();
    $result = [];
    // 启动第一个协程
    $wg->add();
    go(function () use ($wg, &$result) {
        \Swoole\Coroutine\System::sleep(10);
        $result['a'] = 1;
        $wg->done();
    });
    // 启动第二个协程
    $wg->add();
    go(function () use ($wg, &$result) {
        \Swoole\Coroutine\System::sleep(5);
        $result['b'] = 2;
        $wg->done();
    });
    // 挂起当前协程，等待所有任务完成后恢复
    $wg->wait();
    // 这里$result包含了2个任务执行结果
    var_dump($result);
});
```

add()方法增加计数，done()表示任务已完成，wait()等待所有任务完成恢复当前协程的执行，WaitGroup对象可以复用，add、done、wait之后可以再次使用。