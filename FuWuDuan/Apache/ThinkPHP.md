## 在Apache上部署ThinkPHP框架

### 在Apache2.4上部署ThinkPHP5的路由访问问题

#### 情况描述

Windows环境下，在Apache2.4上部署ThinkPHP5时，会出现localhost/public路由可以访问。但是 localhost/public/index路由不可访问的情况。

#### 解决方案

找到Apache的安装目录下的conf文件夹下的httpd.conf文件。打开文件，找到`# LoadModule rewrite_module modules/mod_rewrite.so`这一段代码，如果有`#`号，把`#`号删掉。如果还是不行，找到下面这一段。把`AllowOverride none`改为`AllowOverride all`试试。

```
<Directory />
    AllowOverride none
    Require all denied
</Directory>
```

---

| 参考来源                                                     |
| ------------------------------------------------------------ |
| [ThinkPHP路径与Apache配置](https://blog.csdn.net/littlebo01/article/details/8837230) |
| [windows中，apache/wamp 不能正常访问thinkphp5项目](https://blog.csdn.net/festone000/article/details/80409247) |

