## 在 Larave 中查看框架构造的 Sql 语句

```php
DB::enableQueryLog();
$user = User::all();
return response()->json(DB::getQueryLog());
```

## 如何避免 Sql 注入

```php
User::where('name', '=', $name)->first(); // Eloquent 内部使用 PDO 参数绑定
User::whereRaw("name = '{$name}'")->first(); // 这样的写法需要注意 SQL 注入
User::whereRaw('name = ?', [$name])->first(); // 传参的写法可以避免 SQL 注入的风险
```

## laravel 判断查询结果为空

判断查询结果为空时需要注意 `empty()` 的使用。如果一个查询构造器返回为空，即 `Model::where()->get()` 的结果是空数组。这个时候查询结果集为空，但是返回了查询构造器，所以使用 `empty()` 会返回 `false`。而使用 `first()` 即 `Model::where()->first();` 时，这个时候会从空的查询结果集取第一条数据，这个时候取出的是 `null`，所以使用 `empty()` 会返回 `true`。

```php
empty(Model::where()->get()) = false
count(Model::where()->get()) = 0
empty(Model::where()->first()) = true
count(Model::where()->first()) = 0
```

## 在 laravel 对查询结果集进行分块处理

如果处理的查询结果集过大时，不分块可能会导致内存占用过多。如果你需要处理数千条数据库记录，可以考虑使用 chunk 方法对数据进行分块。该方法每次只取出一小块结果，并将取出的结果传递给闭包处理。顺带一提，使用 `chunk()` 时一定要排序，而且建议使用主键 `id` 进行排序。

```php
DB::table('users')->orderBy('id')->chunk(100, function ($users) {
    foreach ($users as $user) {
        ...
    }
});
```

分块的使用过程中会出现数据处理不完整的问题，这个是由于数据被更新导致的。我们以下面的场景为例。例如，我们需要从数据库中筛选出 1000 条用户信息并删除。

```php
DB::table('users')->orderBy('id')->chunk(100, function ($users) {
    foreach ($users as $user) {
        通过 id 删除用户
    }
});
```

`chunk()` 的底层逻辑是：第一次，取出所有的数据排序并取出第一页，即 id = 1-100 条数据进行删除操作。第二次，取出所有的数据排序，这个时候取出的是 id = 101-1000 的数据，但是现在 chunk 认为第一次已经处理了第一页的数据。这个时候 chunk 取出的是逻辑上的第二页，也就是现在数据库中 id = 201-300 的数据。这样就导致 id = 101-200 的数据没有被处理，也就导致了数据处理不完整。

## Laravel 中检测是否在命令行执行

```php
if (app()->runningInConsole()) {
    // 运行在命令行下
}
```

当然，也可以使用 `PHP` 自身提供的方法：

```php
if (php_sapi_name() == 'cli') {
    // 运行在命令行下
}
```

或者这样：

```php
if (strpos(php_sapi_name(), 'cli') !== false) {
    // 运行在命令行下
}
```

## 获取当前访问 URL

如果控制器方法本身接收 `Request`，可以使用 `url()` 方法。

```php
public function show(Request $request) 
{
     $url = $request->url();
}
```

如果控制器方法本身不接收 `Request`，可以使用下面三种方法。

```php
$url = Request::getRequestUri();
$url = URL::current();
$url = Input::url();
```
