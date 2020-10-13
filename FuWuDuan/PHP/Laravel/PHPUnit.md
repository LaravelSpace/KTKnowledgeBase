```json
{
  "title": "在 Laravel 中使用 PHPUnit",
  "updated_at": "2020-06-30",
  "updated_by": "KelipuTe",
  "tags": "PHP,Laravel,PHPUnit"
}
```

---

## 在 Laravel 中使用 PHPUnit


PHPUnit 是一个面向 PHP 程序员的测试框架，这是一个 xUnit 的体系结构的单元测试框架。

Laravel 框架默认就支持使用 PHPUnit 进行测试。

#### 使用

Laravel 的 PHPUnit 使用 Composer 安装 ，在项目根目录使用 `vendor\bin\phpunit` 命令来使用 PHPUnit。

可以使用 `vendor\bin\phpunit --version` 命令来检查 PHPUnit 是否可用。

#### 编写 PHPUnit 测试

通常，使用 PHPUnit 框架编写测试脚本是需要继承 `PHPUnit\Framework\TestCase` 类，但是在 Laravel 框架中，框架提供了可供继承的类 `Tests\TestCase`。这个类在最底层依然继承了`PHPUnit\Framework\TestCase` 类，并提供了大量便利的方法以供使用。

#### HTTP 测试

```
use Tests\TestCase;

class UserTest extends TestCase
{
    public function testLogin()
    {
        $uri = '/api/user/login';
        $data = [
            'name'     => 'qwe',
            'identity' => 'qwe@q.com',
            'is_email' => true,
            'password' => 'qqqqqq'
        ];
        $headers = [
            'Content-Type' => 'application/json'
        ];
        $response = $this->postJson($uri, $data, $headers);

        $response->assertStatus(200);
    }
}
```

这个例子模拟一个使用 JSON 数据的 POST 请求。

如果测试类中的方法以 test 开头，那么在执行 PHPUnit 测试时，该方法就会被执行。

路由不需要带域名，只需要 **web.php** 或 **api.php** 中提供的 URI 即可。待发送的 JSON 数据使用数组表示，在底层最终会被 `json_encode()` 方法处理。

### 执行 PHPUnit

如果需要执行所有的测试类，则可以在项目根目录使用 `vendor\bin\phpunit` 命令。如果只需要执行单个测试类，只需在 `phpunit` 命令后面加上需要测试的类：`vendor\bin\phpunit tests\Feature\DemoTest`。

测试完成后会在命令行输出结果，如果有需要也可以将测试结果输出到文件。

全部成功的输出样例：

```
PHPUnit 7.5.16 by Sebastian Bergmann and contributors.

.....    5 / 5 (100%)

Time: 437 ms, Memory: 18.00 MB

OK (5 tests, 5 assertions)
```

包含失败的输出样例：

```
PHPUnit 7.5.16 by Sebastian Bergmann and contributors.

...FF    5 / 5 (100%)

Time: 539 ms, Memory: 20.00 MB

There were 2 failures:

1) Tests\Feature\DemoTest::test4
Expected status code 500 but received 200.
Failed asserting that false is true.

path\vendor\laravel\framework\src\Illuminate\Foundation\Testing\TestResponse.php:151
path\tests\Feature\DemoTest.php:38

2) Tests\Feature\DemoTest::test5
Expected status code 500 but received 200.
Failed asserting that false is true.

path\vendor\laravel\framework\src\Illuminate\Foundation\Testing\TestResponse.php:151
path\tests\Feature\DemoTest.php:41

FAILURES!
Tests: 5, Assertions: 5, Failures: 2.
```

包含失败的输出会在输出中展示具体某个测试的某些断言条件没有满足，同时也会标注断言所在的具体位置。

#### 输出日志

如果想要输出日志可以选择在命令行追加参数 `vendor\bin\phpunit --log-junit /temp/phpunit.xml`。

另外，也可以通过配置 Laravel 项目根目录的 `phpunit.xml` 文件实现输出日志。通过配置`<logging>` 元素及其 `<log>` 子元素，记录测试执行期间的日志。

```
<phpunit 
    <logging>
        <log type="junit" target="/temp/log/phpunit.xml"/>
        <log type="testdox-text" target="/temp/log/phpunit.txt"/>
    </logging>
</phpunit>
```

`type` 属性用于配置日志文件的类型，`target` 属性用于配置日志文件的地址。