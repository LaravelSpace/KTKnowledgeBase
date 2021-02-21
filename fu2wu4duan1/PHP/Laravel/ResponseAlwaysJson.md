```json
{
  "title": "让 Laravel 永远返回 Json 格式的响应",
  "updated_at": "2020-06-30",
  "updated_by": "KelipuTe",
  "tags": "PHP,Laravel,Json"
}
```

---

## 让 Laravel 永远返回 Json 格式的响应

在编写完全为 API 服务的 Laravel 应用时，如果希望所有响应都是 JSON 格式的，可以使用下面的两种思路实现。

#### 中间件

首先，创建中间件 JsonMiddleware，JsonMiddleware.php 文件可以放在 app/Http/Middleware 目录下。

```php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class JsonMiddleware 
{
    public function handle(Request $request, Closure $next) 
    {
        $request->headers->set('Accept', 'application/json');

        return $next($request);
    }
}
```

然后，在 **app/Http/Kernel.php** 中添加全局中间件。

```php
    protected $middleware = [
        \App\Http\Middleware\JsonMiddleware::class,
        ...
    ];
```

#### 黑科技

首先，构建一个 `BaseRequest` 继承 `Illuminate\Http\Request`，并修改为默认优先使用 JSON 响应。BaseRequest.php 文件可以放在 app/Http/Requests 目录下。

```php
namespace App\Http\Requests;

use Illuminate\Http\Request;

class BaseRequest extends Request
{
    public function expectsJson()
    {
        return true;
    }
    
    public function wantsJson()
    {
        return true;
    }
}
```

然后 ，在 public/index.php 中，将 `\Illumiate\Http\Request` 替换为刚才重写 `BaseRequest`。

```php
$response = $kernel->handle(
    $request = \App\Http\Requests\BaseRequest::capture()
);
```

现在所有的响应都是 `application/json`，包括错误和异常。

---

## 参考

[如何让你的 Laravel API 永远返回 JSON 格式的响应？](https://laravel-china.org/wikis/16069)