## 安装MySQL

### 在Windows中安装MySQL

##### 下载MySQL

这里用的是解压版的mysql-5.7.18。

##### 安装MySQL

解压mysql-5.7.18-winx64.rar到合适的位置。

##### 添加系统环境变量

新建变量：MYSQL_HOME=>...\mysql-5.7.18-winx64（你的MySQL根目录）

追加变量：PATH=>`%MYSQL_HOME%\bin`（你的MySQL根目录下bin文件夹）

##### 配置my.ini文件

在mysql的根目录新建my.ini文件。

```
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8

[mysqld]
#这个先不要，这是忽略权限
#skip_grant_tables
#设置3306端口
port = 3306
# 设置mysql的安装目录
basedir=C:\Learning\MySQL\mysql-5.7.18-winx64
# 设置mysql数据库的数据的存放目录
datadir=C:\Learning\MySQL\mysql-5.7.18-winx64\data
# 允许最大连接数
max_connections=200
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
```

##### 新增data文件夹

在mysql根目录下新建data文件夹

##### 安装MySQL服务

用命令行进入根目录下的bin目录，输入`mysqld install`命令。出现安装成功说明就OK了。

##### 启动MySQL服务器

`我的电脑=>右击=>管理=>服务和应用程序=>服务`，找到MySQL服务并启动。

如果发生下面的报错，就先清空mysql根目录下的data文件夹。然后，用命令行进入bin目录，输入`mysqld --initialize`命令执行初始化。最后，在设备管理器服务管理内再次启动mysql服务器。

> 本地计算机上的 MySQL 服务启动后停止。某些服务在未由其他服务或程序使用时将自动停止？

##### 修改登录密码

在mysql根目录下data文件夹内，找到**DESKTOP-P1770NP.err**文件。（.err文件的名字每台电脑不一样）用记事本打开.err文件找到如下内容：

```
2017-04-17T04:50:22.777490Z 1 [Note] A temporary password is generated for root@localhost: &Ga/k>jYy2Z1
```

用命令行登录mysql服务： `mysql -u root -p`。`&Ga/k>jYy2Z1` 就是mysql在安装时，随机生成的root账号的密码。可以用命令修改密码为new_password：`mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';`。还可以用命令`set password for root@localhost = password('new_password');`修改密码。

### MySQL8.0

Navicat连接MySQL报错：

```
Authentication plugin 'caching_sha2_password' cannot be loaded
```

##### 解决方案1

适用场景：第一次构建容器/安装、已安装完成后新增用户

配置mysql.cnf配置默认身份验证插件：

```
[mysqld]
default_authentication_plugin = mysql_native_password
```

##### 解决方案2

适用场景：MySQL已成功安装完成后

查看身份验证类型：

```
mysql> use mysql;
Database changed

mysql> SELECT Host, User, plugin from user;
+-----------+------------------+-----------------------+
| Host      | User             | plugin                |
+-----------+------------------+-----------------------+
| %         | root             | caching_sha2_password |
| localhost | mysql.infoschema | caching_sha2_password |
| localhost | mysql.session    | caching_sha2_password |
| localhost | mysql.sys        | caching_sha2_password |
| localhost | root             | caching_sha2_password |
+-----------+------------------+-----------------------+
5 rows in set (0.00 sec)
```

发现root用户的验证器插件为caching_sha2_password。

修改身份验证类型(同时会修改密码)。

```
mysql> ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
Query OK, 0 rows affected (0.00 sec)

mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';
Query OK, 0 rows affected (0.01 sec)
```


让设置生效

```
mysql> FLUSH PRIVILEGES;
```


验证是否生效

```
mysql> SELECT Host, User, plugin from user;
+-----------+------------------+-----------------------+
| Host      | User             | plugin                |
+-----------+------------------+-----------------------+
| %         | root             | mysql_native_password |
| localhost | mysql.infoschema | caching_sha2_password |
| localhost | mysql.session    | caching_sha2_password |
| localhost | mysql.sys        | caching_sha2_password |
| localhost | root             | mysql_native_password |
+-----------+------------------+-----------------------+
5 rows in set (0.00 sec)
```


