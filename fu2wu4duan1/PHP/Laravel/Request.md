```json
{
  "title": "Laravel 的 Illuminate\Http\Request 类",
  "updated_at": "2020-06-30",
  "updated_by": "KelipuTe",
  "tags": "PHP,Laravel,Request"
}
```

---

## Laravel 的 Illuminate\Http\Request 类

接收请求的时候，传入的请求实例将会由 Laravel 的服务容器自动注入。要通过依赖注入获取当前 HTTP 请求实例，需要在控制器上引入 **Illuminate\Http\Request** 类。这个类是 Laravel 请求的核心类，基本上每一个请求都会用到。

#### 获取当前访问 URL

如果控制器方法本身接收 Illuminate\Http\Request，则可以直接使用 url() 方法。

但是，如果控制器方法本身不接收 Illuminate\Http\Request，则可以使用 Illuminate\Support\Facades\URL 类的 current() 方法，也可以简写成 \URL::current()。

#### 操作请求的原始数据

使用 Request::all() 可以获取到所有的请求参数构成的数组。如果想要请求中的一部分参数，则可以使用 Request::only() 获取指定的参数构成的数组。或者使用 Request::except() 移除不需要的参数，返回由剩余的参数构成的数组。

但是 Request::only() 和 Request::except() 两个方法并不会对原始数据进行修改。如果需要在中间件层，过滤掉无用的参数，或者追加新的参数，就需要对请求的原始数据进行修改。这时就需要使用 Request::offsetSet() 和 Request::offsetUnset() 两个方法。

如果查看源码的话不拿发现 Request::offsetSet() 和 Request::offsetUnset() 两个方法都调用了 Request::getInputSource() 得到请求的原始参数。

```php
protected function getInputSource()
    {
        if ($this->isJson()) {
            return $this->json();
        }

        return in_array($this->getRealMethod(), ['GET', 'HEAD']) ? $this->query : $this->request;
    }
```

这里请求的原始数据又分了三个来源 GET 请求、非 GET 请求 和 Json 请求。
Illuminate\Http\Request 继承了 Symfony\Component\HttpFoundation\Request。上面代码片段中的 `$this->query` 和 `$this->request` 都可以直接找到。都是 Symfony\Component\HttpFoundation\ParameterBag 的实例。Json 请求的参数则相对复杂。

上面代码片段中的 `$this->query` 和 `$this->request` 都可以在 Illuminate\Http\Request 中直接使用。如果在控制器里，则可以通过 `$request->query` 和 `$request->request` 直接调用。

再给 Laravel 项目发 json 请求的时候要带上 header，要不然可能会解析错误。