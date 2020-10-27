## GET与POST的区别

首先给一个参考答案，来自w3school：[HTTP 方法：GET 对比 POST](https://www.w3school.com.cn/tags/html_ref_httpmethods.asp)。

| 操作             | GET                                                          | POST                                                         |
| :--------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 后退按钮/刷新    | 无害                                                         | 数据会被重新提交（浏览器应该告知用户数据会被重新提交）。     |
| 书签             | 可收藏为书签                                                 | 不可收藏为书签                                               |
| 缓存             | 能被缓存                                                     | 不能缓存                                                     |
| 编码类型         | application/x-www-form-urlencoded                            | application/x-www-form-urlencoded 或 multipart/form-data。为二进制数据使用多重编码。 |
| 历史             | 参数保留在浏览器历史中。                                     | 参数不会保存在浏览器历史中。                                 |
| 对数据长度的限制 | 是的。当发送数据时，GET 方法向 URL 添加数据；URL 的长度是受限制的（URL 的最大长度是 2048 个字符）。 | 无限制。                                                     |
| 对数据类型的限制 | 只允许ASCII字符。                                            | 没有限制。也允许二进制数据。                                 |
| 安全性           | 与POST相比，GET的安全性较差，因为所发送的数据是URL的一部分。在发送密码或其他敏感信息时绝不要使用GET ！ | POST比GET更安全，因为参数不会被保存在浏览器历史或web服务器日志中。 |
| 可见性           | 数据在URL中对所有人都是可见的。                              | 数据不会显示在URL中。                                        |

首先这里要说明，不是这个参考答案有问题，而是这个参考答案只是针对GET和POST的比较，比较的范围很狭小。不要认为这个比较的结果，在所有的场景都成立。

### 从报文的角度

在网络中，HTTP只是个行为准则，真正用来传输数据的是TCP。所以从报文的角度，GET和POST没有实质的区别，GET和POST只是HTTP协议中两种请求方式，而HTTP协议是基于TCP/IP的应用层协议，无论GET还是POST，用的都是同一个传输层协议，所以在传输上，没有区别。

报文格式上，不带参数时，最大区别就是第一行方法名不同。

```http
GET /uri HTTP/1.1 \r\n
```

```http
POST /uri HTTP/1.1 \r\n
```

在约定中带参数时，GET方法的参数应该放在url中，POST方法参数应该放在body中。

```http
GET /index.php?name=xxx&age=18 HTTP/1.1
Host: localhost
```

```http
POST /index.php HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded

name=xxx&age=18
```

但是这也只是约定，是可以不遵守的。只要服务端能把数据解析出来，改成下面这样也是可以的。

```http
GET /index.php/name@xxx/age@18 HTTP/1.1
```

```http
POST /index.php/name@xxx/age@18 HTTP/1.1
```

再比如，ElasticSearch的HTTPAPI：[ElasticSearch的HTTPAPI](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-body.html)，就允许使用GET方法提交request body。

> Both HTTP GET and HTTP POST can be used to execute search with body. Since not all clients support GET with body, POST is allowed as well.

### 从安全的角度

HTTP协议的GET和POST都不安全，因为他们的传输都是明文的。想要安全传输那就必须加密，也就是用HTTPS。

### GET方法的数据长度限制

HTTP协议没有url和body的长度限制，大多数对url的限制都是浏览器和服务器，因为数据量太大对浏览器和服务器都是很大负担。

### POST产生两个TCP数据包

HTTP协议中没有明确的说明POST会产生两个TCP数据包。header和body分开发送，可以认为是自发的行为。比如，在使用curl做POST的时候，当要POST的数据大于1024字节的时候，curl并不会直接就发起POST请求，而是会分为俩步：1、发送一个请求，包含一个Expect:100-continue，询问Server使用愿意接受数据。2、接收到Server返回的100-continue应答以后, 才把数据POST给Server。并不是所有的Server都会正确应答100-continue，比如lighttpd，就会返回417“Expectation Failed”，则会造成逻辑出错。