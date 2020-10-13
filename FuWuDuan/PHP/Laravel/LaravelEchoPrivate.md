```json
{
  "title": "在 Laravel Echo 中使用私有频道",
  "updated_at": "2020-06-30",
  "updated_by": "KelipuTe",
  "tags": "PHP,Laravel,Laravel Echo"
}
```

---

## 在 Laravel Echo 中使用私有频道

这里介绍使用中间件进行 Token 认证的方式，授权私有频道。

#### 监听私有频道

```javascript
window.Echo.private("broadcast-user.13")
    .listen(".private-user", (e) => {
        console.log(".private-user");
        console.log(e);
    })
```

在上一篇里，已经介绍了，怎么监听公有频道。监听私有频道就把 `window.Echo.channel()` 改成 `window.Echo.private()`。区别是，监听私有频道时，Echo 服务会先发送一个 post 认证请求到 **.../broadcasting/auth** 路由进行授权认证。

如果认证成功则可以监听私有频道，示例输出如下。

```
[18:49:30] - Preparing authentication request to: http://localhost:8000
[18:49:30] - Sending auth request to: http://localhost:8000/broadcasting/auth

[18:49:30] - dDphamXJxvBswY7jAAAI authenticated for: private-broadcast-user.13
[18:49:30] - dDphamXJxvBswY7jAAAI joined channel: private-broadcast-user.13
```

如果认证失败则会输出失败信息，示例输出如下。

```
[18:49:13] - Preparing authentication request to: http://localhost:8000
[18:49:13] - Sending auth request to: http://localhost:8000/broadcasting/auth

? [18:49:13] - bhsAQ2WeLuVP5T06AAAH could not be authenticated to private-broadcast-user.1
{"status":"error","data":[],"message":"\u9891\u9053\u6388\u6743\u5931\u8d25"}
Client can not be authenticated, got HTTP status 400
```

#### 修改前端 Echo 服务配置

```javascript
window.Echo = new Echo({
    broadcaster: 'socket.io',
    host: window.location.hostname + ':6001',
    auth: {
        headers:
            {
                'authorization': ''
            }
    }
});
```

Echo 服务的配置需要修改一下，为请求加上请求头，这样中间件就可以使用 Token 认证用户。

#### 新建中间件

```php
class ChannelAuth
{
    public function handle(Request $request, Closure $next)
    {
        $authorization = $request->header('authorization', '');
        if (is_string($authorization) && $authorization !== '') {
            ...
            // 这里省略具体的 Token 认证逻辑
            \Auth::loginUsingId((int)$clientId);
            $response = $next($request);
        } else {
            $response = response('false', 400);
        }

        return $response;
    }
}
```

中间件在这里要做两件事，一是进行 Token 认证，二是 Laravel 私有频道需要用到自带的 Auth 用户认证，所以 Token 认证后，要使用 Laravel 的 Auth 登录用户。

认证成功返回 HTTP 状态码 200 即可。如果认证失败，返回 400、500 这样的 HTTP 状态码就可以让 Echo 服务发起的认证请求失败。

这里具体内容需要参考官方文档：[Laravel 中文文档-广播系统](https://learnku.com/docs/laravel/5.6/broadcasting/1386#authorizing-channels)

如果要启用中间件，需要修改 **BroadcastServiceProvider** 类。在 `Broadcast::routes()` 里添加中间件。这里的用法和路由那里是一样的。

```php
public function boot()
    {
        Broadcast::routes(['middleware' => 'middleware_name']);

        require base_path('routes/channels.php');
    }
```

最后可以选择在 **channel.php** 文件里处理具体的授权逻辑，也可以在中间件里直接处理，实际上效果是一样的。

```php
Broadcast::channel('broadcast-user.{id}', function ($user, $id) {
    return (int) $user->id === (int) $id;
});
```

这里文档里描述的是返回 true 或者 false 就可以表示授权成功或者失败。具体实现时，返回 true 是没问题的。但是返回 false 的时候，实际上是触发了一个 **AccessDeniedHttpException** 异常。

---

#### 参考

[Laravel 中文文档-广播系统](https://learnku.com/docs/laravel/5.6/broadcasting/1386#authorizing-channels)

[使用 Laravel-echo-server 构建实时应用（二）私有频道 ](https://learnku.com/laravel/t/13521/using-laravel-echo-server-to-build-real-time-applications-two-private-channels)

[深入浅出 Laravel Echo ](https://learnku.com/articles/17327)

[由浅入深：基于 Laravel Broadcast 实现 WebSocket C/S 实时通信](https://xueyuanjun.com/index.php/post/8559.html#bkmrk-0x00-%E5%87%86%E5%A4%87)