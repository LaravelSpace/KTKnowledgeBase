## 在 Apache2.4 上部署 ThinkPHP5 的路由访问问题

#### 情况描述

Windows 环境下，在 Apache2.4 上部署 ThinkPHP3.2 时，会出现 localhost/public 可以访问。但是 localhost/public/index/index 不可访问的情况。

#### 解决方案

找到 Apache 的安装目录下的 conf 文件夹下的 httpd.conf 文件。

打开文件，找到 `# LoadModule rewrite_module modules/mod_rewrite.so` 这一段代码，如果有#，把 # 号删掉。

如果还是不行，找到下面这一段。把 none 改为 all 试试。

```
<Directory />
    AllowOverride none
    Require all denied
</Directory>
```

## 参考

[ThinkPHP路径与Apache配置](https://blog.csdn.net/littlebo01/article/details/8837230)

[windows中，apache/wamp 不能正常访问thinkphp5项目](https://blog.csdn.net/festone000/article/details/80409247)