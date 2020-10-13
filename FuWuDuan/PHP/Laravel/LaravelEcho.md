```json
{
  "title": "使用 Laravel Echo 构建实时应用",
  "updated_at": "2020-06-30",
  "updated_by": "KelipuTe",
  "tags": "PHP,Laravel,Laravel Echo"
}
```

---

## 使用 Laravel Echo 构建实时应用

#### 安装 laravel-echo-server

在安装 laravel-echo-server 之前先要安装 Node.js。

使用命令 `npm install -g laravel-echo-server` 全局安装 laravel-echo-server，输出示例如下。
```
$ npm install -g laravel-echo-server
npm WARN deprecated request@2.88.2: request has been deprecated, see https://github.com/request/request/issues/3142
C:\Users\...\AppData\Roaming\npm\laravel-echo-server -> C:\Users\...\AppData\Roaming\npm\node_modules\laravel-echo-server\bin\server.js

> spawn-sync@1.0.15 postinstall C:\Users\...\AppData\Roaming\npm\node_modules\laravel-echo-server\node_modules\spawn-sync
> node postinstall

+ laravel-echo-server@1.6.0
added 260 packages from 144 contributors in 287.95s
```

安装完成后，切换到 Laravel 项目目录下。使用命令 `laravel-echo-server init` 初始化 Socket 服务。在初始化过程中需要配置各种参数，具体参数根据实际情况进行配置。输出示例如下。
```
$ laravel-echo-server init
? Do you want to run this server in development mode? (y/N) y
? Do you want to run this server in development mode? Yes
? Which port would you like to serve from? (6001) 6001
? Which port would you like to serve from? 6001
? Which database would you like to use to store presence channel members? redis
? Enter the host of your Laravel authentication server. (http://localhost) http://localhost:8000
? Enter the host of your Laravel authentication server. http://localhost:8000
? Will you be serving on http or https? (Use arrow keys)
? Will you be serving on http or https? http
? Do you want to generate a client ID/Key for HTTP API? (y/N) no
? Do you want to generate a client ID/Key for HTTP API? No
? Do you want to setup cross domain access to the API? (y/N) no
? Do you want to setup cross domain access to the API? No
? What do you want this config to be saved as? (laravel-echo-server.json)
? What do you want this config to be saved as? laravel-echo-server.json
Configuration file saved. Run laravel-echo-server start to run server.
```

初始化完成后，Laravel 项目目录下会多一个 **laravel-echo-server.json** 文件。

如果初始化配置参数时用到的 redis 数据库有密码，则需要在 laravel-echo-server.json 文件中额外进行配置。
```
    "databaseConfig": {
		"redis": {
			"host":"127.0.0.1",
			"port":6379,
			"password":"password"
		},
		"sqlite": {
			"databasePath": "/database/laravel-echo-server.sqlite"
		}
	},
```

配置完成后，使用命令 `$ laravel-echo-server start` 启动服务。示例输出如下。

```
$ laravel-echo-server start

L A R A V E L  E C H O  S E R V E R

version 1.6.0

? Starting server in DEV mode...

?  Running at localhost on port 6001
?  Channels are ready.
?  Listening for http events...
?  Listening for redis events...

Server ready!
```

启动服务后窗口不要关闭。

#### 配置 Laravel 项目

取消 **config/app.php** 文件中 BroadcastServiceProvider 在 Providers 数组中的注释。

```php
/*
 * Application Service Providers...
 */
App\Providers\AppServiceProvider::class,
App\Providers\AuthServiceProvider::class,
// BroadcastServiceProvider 原来是被注释的
// App\Providers\BroadcastServiceProvider::class, 
App\Providers\EventServiceProvider::class,
App\Providers\RouteServiceProvider::class,
```

打开 **.env** 文件，修改 `BROADCAST_DRIVER=` 的值为  laravel-echo-server 初始化时定义的值（上文定义的值是 redis）

#### 安装 Socket.io 客户端和 Laravel-Echo 包

在 Laravel 项目目录下，执行命令 `$ npm install` 安装相关依赖。

使用命令 `$ npm install --save socket.io-client` 和 `$ npm install --save laravel-echo` 分别安装 Socket.io 客户端和 Laravel-Echo 包。实例输出如下

```
$ npm install --save socket.io-client
npm notice created a lockfile as package-lock.json. You should commit this file.
+ socket.io-client@2.3.0
added 30 packages from 22 contributors and audited 48 packages in 12.979s
found 0 vulnerabilities
```

```
$ npm install --save laravel-echo
+ laravel-echo@1.6.1
added 1 package from 1 contributor and audited 49 packages in 4.423s
found 0 vulnerabilities
```

在 **resources/assets/js/bootstrap.js**  文件里，Laravel 已经提供了基础的 Echo 服务代码。

```javascript
// import Echo from 'laravel-echo'

// window.Pusher = require('pusher-js');

// window.Echo = new Echo({
//     broadcaster: 'pusher',
//     key: process.env.MIX_PUSHER_APP_KEY,
//     cluster: process.env.MIX_PUSHER_APP_CLUSTER,
//     encrypted: true
// });
```

可以直接用这里的代码，也可以自己新建一个 js 文件来写。如果不是全局都要使用 Echo 服务，建议新建一个 js 来写。哪里需要，就在哪里引入。新建 laravel-echo.js 文件，添加启动 Echo 服务的代码。

```javascript
import Echo from 'laravel-echo'

window.io = require('socket.io-client');
window.Echo = new Echo({
    broadcaster: 'socket.io',
    host: window.location.hostname + ':6001'
});
```

新建完成后，使用 Laravel 提供的前端编译工具 Mix 进行编译。首先打开 **webpack.mix.js** 添加如下代码：`mix.js('resources/js/laravel-echo.js', 'public/js');`。这表示将 resources/js/laravel-echo.js 文件编译后生成的的 js 文件放到 public/js 目录下。

完成后使用命令 `npm run dev` 进行编译。示例输出如下。

```
$ npm run dev

> @ dev E:\Workspace\PHPWorkspace\WarshipCommunity
> npm run development


> @ development E:\Workspace\PHPWorkspace\WarshipCommunity
> cross-env NODE_ENV=development node_modules/webpack/bin/webpack.js --progress --hide-modules --config=node_modules/laravel-mix/setup/webpack.config.js

 10% building modules 0/1 modules 1 active ...ommunity\resources\js\laravel_echo 10% building modules 1/2 modules 1 active ...ommunity\resources\js\laravel_echo 10% building modules 2/3 modules 1 active ...ode_modules\laravel-echo\dist\echo 10% building modules 3/4 modules 1 active ...modules\socket.io-client\lib\index 10% building modules 4/5 modules 1 active ...e_modules\socket.io-client\lib\url 10% building modules 4/6 modules 2 active ...dules\socket.io-client\lib\manager 10% building modules 4/7 modules 3 active ...odules\socket.io-client\lib\socket 10% building modules 5/7 modules 2 active ...odules\socket.io-client\lib\socket 10% building modules 6/7 modules 1 active ...odules\socket.io-client\lib\socket 10% building modules 7/8 modules 1 active ...de_modules\socket.io-client\lib\on 10% building modules 7/9 modules 2 active ...ity\node_modules\debug\src\browser 10% building modules 7/10 modules 3 active ...ode_modules\socket.io-parser\inde 10% building modules 8/10 modules 2 active ...ode_modules\socket.io-parser\inde 11% building modules 9/10 modules 1 active ...ode_modules\socket.io-parser\inde 11% building modules 10/11 modules 1 active ...de_modules\component-emitter\ind 11% building modules 10/12 modules 2 active ...nity\node_modules\debug\src\comm 11% building modules 10/13 modules 3 active ...modules\engine.io-client\lib\ind 11% building modules 10/14 modules 4 active ...unity\node_modules\process\brows 11% building modules 10/15 modules 5 active ...munity\node_modules\parseuri\ind 11% building modules 10/16 modules 6 active ...\node_modules\component-bind\ind 11% building modules 11/16 modules 5 active ...\node_modules\component-bind\ind 11% building modules 11/17 modules 6 active ...de_modules\socket.io-parser\bina 11% building modules 11/18 modules 7 active ...modules\socket.io-parser\is-buff 11% building modules 11/19 modules 8 active ...ommunity\node_modules\backo2\ind 11% building modules 11/20 modules 9 active ...ity\node_modules\has-binary2\ind 11% building modules 11/21 modules 10 active ...munity\node_modules\to-array\in 11% building modules 12/21 modules 9 active ...munity\node_modules\to-array\ind 11% building modules 13/21 modules 8 active ...munity\node_modules\to-array\ind 11% building modules 14/21 modules 7 active ...munity\node_modules\to-array\ind 11% building modules 14/22 modules 8 active ...mmunity\node_modules\parseqs\ind 11% building modules 14/23 modules 9 active ...mmunity\node_modules\indexof\ind 11% building modules 15/23 modules 8 active ...mmunity\node_modules\indexof\ind 11% building modules 16/23 modules 7 active ...mmunity\node_modules\indexof\ind 12% building modules 17/23 modules 6 active ...mmunity\node_modules\indexof\ind 12% building modules 18/23 modules 5 active ...mmunity\node_modules\indexof\ind 12% building modules 19/23 modules 4 active ...mmunity\node_modules\indexof\ind 12% building modules 20/23 modules 3 active ...mmunity\node_modules\indexof\ind 12% building modules 21/23 modules 2 active ...mmunity\node_modules\indexof\ind 12% building modules 22/23 modules 1 active ...mmunity\node_modules\indexof\ind 12% building modules 23/24 modules 1 active ...odules\engine.io-client\lib\sock 12% building modules 23/25 modules 2 active ...mmunity\node_modules\isarray\ind 12% building modules 23/26 modules 3 active ...ommunity\node_modules\buffer\ind 12% building modules 24/26 modules 2 active ...ommunity\node_modules\buffer\ind 12% building modules 24/27 modules 3 active ...ser\node_modules\debug\src\brows 13% building modules 25/27 modules 2 active ...ser\node_modules\debug\src\brows 13% building modules 26/27 modules 1 active ...ser\node_modules\debug\src\brows 13% building modules 27/28 modules 1 active ...dules\engine.io-parser\lib\brows 13% building modules 27/29 modules 2 active ...les\engine.io-client\lib\transpo 13% building modules 27/30 modules 3 active ...ine.io-client\lib\transports\ind 13% building modules 27/31 modules 4 active ...hipCommunity\node_modules\ms\ind 13% building modules 27/32 modules 5 active ...ode_modules\webpack\buildin\glob 13% building modules 28/32 modules 4 active ...ode_modules\webpack\buildin\glob 13% building modules 29/32 modules 3 active ...ode_modules\webpack\buildin\glob 13% building modules 30/32 modules 2 active ...ode_modules\webpack\buildin\glob 13% building modules 30/33 modules 3 active ...arser\node_modules\debug\src\deb 13% building modules 31/33 modules 2 active ...arser\node_modules\debug\src\deb 13% building modules 32/33 modules 1 active ...arser\node_modules\debug\src\deb 13% building modules 32/34 modules 2 active ...ngine.io-client\lib\xmlhttpreque 13% building modules 33/34 modules 1 active ...ngine.io-client\lib\xmlhttpreque 13% building modules 33/35 modules 2 active ..._modules\engine.io-parser\lib\ut 13% building modules 33/36 modules 3 active ..._modules\engine.io-parser\lib\ke 14% building modules 34/36 modules 2 active ..._modules\engine.io-parser\lib\ke 14% building modules 34/37 modules 3 active ...-client\lib\transports\polling-x 14% building modules 34/38 modules 4 active ...lient\lib\transports\polling-jso 14% building modules 34/39 modules 5 active ...io-client\lib\transports\websock 14% building modules 34/40 modules 6 active ...unity\node_modules\base64-js\ind 14% building modules 34/41 modules 7 active ...mmunity\node_modules\ieee754\ind 14% building modules 35/41 modules 6 active ...mmunity\node_modules\ieee754\ind 14% building modules 35/42 modules 7 active ...\buffer\node_modules\isarray\ind 14% building modules 36/42 modules 6 active ...\buffer\node_modules\isarray\ind 14% building modules 37/42 modules 5 active ...\buffer\node_modules\isarray\ind 14% building modules 38/42 modules 4 active ...\buffer\node_modules\isarray\ind 14% building modules 39/42 modules 3 active ...\buffer\node_modules\isarray\ind 14% building modules 40/42 modules 2 active ...\buffer\node_modules\isarray\ind 14% building modules 40/43 modules 3 active ...es\engine.io-client\lib\transpor 14% building modules 41/43 modules 2 active ...\buffer\node_modules\isarray\ind 15% building modules 42/43 modules 1 active ...\buffer\node_modules\isarray\ind 15% building modules 43/44 modules 1 active ...e.io-client\lib\transports\polli 15% building modules 43/45 modules 2 active ...Community\node_modules\after\ind 15% building modules 43/46 modules 3 active ...de_modules\arraybuffer.slice\ind 15% building modules 43/47 modules 4 active ...pCommunity\node_modules\blob\ind 15% building modules 43/48 modules 5 active ...arraybuffer\lib\base64-arraybuff 15% building modules 44/48 modules 4 active ...arraybuffer\lib\base64-arraybuff 15% building modules 45/48 modules 3 active ...arraybuffer\lib\base64-arraybuff 15% building modules 46/48 modules 2 active ...arraybuffer\lib\base64-arraybuff 15% building modules 47/48 modules 1 active ...arraybuffer\lib\base64-arraybuff 15% building modules 48/49 modules 1 active ...et.io-parser\node_modules\ms\ind 15% building modules 48/50 modules 2 active ...munity\node_modules\has-cors\ind 15% building modules 49/50 modules 1 active ...munity\node_modules\has-cors\ind 16% building modules 50/51 modules 1 active ...Community\node_modules\yeast\ind 16% building modules 50/52 modules 2 active ...de_modules\component-inherit\ind 16% building modules 51/52 modules 1 active ...de_modules\component-inherit\ind 95% emitting DONE  Compiled successfully in 663ms13:47:04

                        Asset    Size  Chunks                    Chunk Names
/laravel_echo/laravel_echo.js  254 kB       0  [emitted]  [big]  /laravel_echo/laravel_echo

```

#### 在 Laravel 中使用广播系统

在 **app/Events** 目录下面创建一个 **NotificationEvent** 事件类，继承 ShouldBroadcast 接口。

找到 `broadcastOn()` 函数，设置频道名称。

```php
public function broadcastOn()
{
    return new Channel('test-event');
}
```

为了方便测试，找到 `broadcastWith()` 函数，添加一些数据。

```php
public function broadcastWith()
{
    return [
        'data' => 'aaa'
    ];
}
```

找个地方，使用 `broadcast()` 方法广播这个事件。

```php
broadcast(new \App\Events\NotificationEvent);
```

执行过这里的代码后，如果没有定义任务执行的队列，则会生成一条任务分发到默认队列 default 去。执行命令 `php artisan queue:work`，处理队列中的任务。这时会在刚才启动 laravel-echo-server 的界面看见如下输出。

```
Channel: test-event
Event: App\Events\NotificationEvent
```

#### 前端监听频道和事件

首先引入编译好的 laravel-echo.js 文件。接着编写监听代码。

```javascript
window.Echo.channel('test-event')
    .listen('NotificationEvent', (e) => {
        console.log(e);
    });
```

这表示监听 test-event 频道的 NotificationEvent 事件，然后输出到控制台。

如果代码正常工作的话，在打开页面开始监听和关闭页面退出监听时，在刚才启动 laravel-echo-server 的界面可以分别看到如下的输出。

```
[14:19:21] - FwuGtdb1SB3Nfup9AAAH joined channel: test-event

[14:19:20] - TNJTGWVN9NmEEqy7AAAG left channel: test-event (transport close)
```

另外需要注意的是，如果 Laravel 项目里，事件不是在 **app/Events** 目录下的而是在其他地方，比如 `App\Events\Community\NotificationEvent` 这样的。那么前端监听事件时，也要相应地变成：`listen('Community\\NotificationEvent', (e) => {})`。

---

#### 参考

[Laravel 中文文档-广播系统](https://learnku.com/docs/laravel/5.6/broadcasting/1386#authorizing-channels)

[使用 Laravel-echo-server 构建实时应用](https://learnku.com/laravel/t/13101/using-laravel-echo-server-to-build-real-time-applications)

[laravel5.5事件广播系统实例laravel-echo + redis + socket.io](https://www.cnblogs.com/redirect/p/8658800.html)