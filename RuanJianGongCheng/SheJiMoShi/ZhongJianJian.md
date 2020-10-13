```json
{
  "title": "中间件（Middleware）",
  "updated_at": "2020-05-21",
  "updated_by": "KelipuTe",
  "tags": "设计模式,中间件,Middleware"
}
```

---

## 中间件（Middleware）

这个设计模式由 PHP 编码实现。

#### Laravel 中间件

Laravel 的中间件可以想象成一个套娃一样的处理流程。每增加一个功能就多套一层，新的层可以安插在原来任意两个层之间。从而达到不修改已有代码的情况下，扩展程序功能的目的。

有点像面向切片编程（Aspect Oriented Programming，即 AOP）的程序处理流程。想象成一个类似洋葱的结构，一层一层进去，再一层一层出来。值得注意的是，进入时的第一层，在出来的时候，是最后一层。

下面是实现代码：

```php
interface Middleware
{
    public static function handle(Closure $next);
}

class VerifyCsrfToekn implements Middleware
{
    public static function handle(Closure $next)
    {
        echo "验证 CSRF Token。" . PHP_EOL;
        $next();
    }
}

class VerifyAuth implements Middleware
{
    public static function handle(Closure $next)
    {
        echo "验证是否登录。" . PHP_EOL;
        $next();
    }
}

class CookieHandler implements Middleware
{
    public static function handle(Closure $next)
    {
        echo "获取 Cookie 信息。" . PHP_EOL;
        $next();
        echo "设置 Cookie 信息。" . PHP_EOL;
    }
}

$handle = function () {
    echo "要执行的程序。" . PHP_EOL;
};

/* 顺序调用的方式 */

function call_middleware($handle)
{
    VerifyCsrfToekn::handle(function () use ($handle) {
        VerifyAuth::handle(function () use ($handle) {
            CookieHandler::handle($handle);
        });
    });
}

/* 管道调用的方式 */

// 管道这里在前面的数组元素处理后会在内层
$pipeArray = [
    'CookieHandler',
    'VerifyAuth',
    'VerifyCsrfToekn',
];
// array_reduce(数组，函数，初始值)
// array_reduce() 的作用是使用函数依次处理数组中的值
$callback = array_reduce(
    $pipeArray,
    function ($stack, $pipe) {
        return function () use ($stack, $pipe) {
            return $pipe::handle($stack);
        };
    },
    $handle
);

/* 测试代码 */

echo "顺序调用的方式。" . PHP_EOL;
call_middleware($handle);
echo "管道调用的方式。" . PHP_EOL;
call_user_func($callback);
```

顺序调用的方式，比较容易理解，其结果和 Laravel 中间件执行的结果相同，但是很显然这样的代码不利于维护。

管道调用的方式，看起来比较复杂，而且可以注意到，数组的顺序是反的，这里做一个程序运行流程的解释。

第一次执行 stack=handle()，pipe=CookieHandler。

执行的结果返回了一个函数 CookieHandler::handle(handle())。

第二次执行 stack=CookieHandler::handle(handle())，pipe=VerifyAuth。

执行的结果 VerifyAuth::handle(CookieHandler::handle(handle()))。

依次类推第三次，第四次直到数组中所有的元素都被处理过。

这样就比较容易理解，为什么数组中的元素和中间件执行的顺序是反的了。

最后 callback 是一个回调函数，需要执行才会出结果。`$callback()` 这样也可以。

---

## 参考

[Laravel中间件,管道之面向切面编程](https://learnku.com/docs/laravel-core-concept/5.5/%E4%B8%AD%E9%97%B4%E4%BB%B6/3022)