```json
{
  "title": "在 Windows 中安装 MySql",
  "updated_at": "2020-06-29",
  "updated_by": "KelipuTe",
  "tags": "Windows,Mysql"
}
```

---

﻿## 在 Windows 中安装 MySql

#### 下载 MySql

这里用的是**解压版**的 **mysql-5.7.18**。

#### 安装 MySql

解压 mysql-5.7.18-winx64.rar 到合适的位置

#### 添加系统环境变量

**新建**变量：**MYSQL_HOME** => C:\Learning\MySQL\mysql-5.7.18-winx64（你的MySQL根目录）

**追加**变量：**PATH** => `%MYSQL_HOME%\bin`（你的MySQL根目录下bin文件夹）

#### 配置 **my.ini** 文件

在 mysql 的根目录下面新建一个 my.ini 文件，在 my.ini 文件里面写上如下代码后，保存

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

#### 新增 **data/** 文件夹

在 mysql 根目录下新建一个文件夹命名为 data

#### 安装 MySql 服务

打开 cmd.exe 进入根目录下的 bin 目录输入 `mysqld install`。出现安装成功说明就OK了。

#### 启动 MySql 服务器

在 **我的电脑==>右击==>管理==>服务和应用程序==>服务** 界面找到 MySQL 服务并启动。

如果出现问题：

```
本地计算机上的 MySQL 服务启动后停止。某些服务在未由其他服务或程序使用时将自动停止？
```

将 mysql 的根目录下 data 文件夹清空。然后，打开 cmd.exe 进入 bin 目录输入 `mysqld --initialize` 执行初始化。在设备管理器服务管理内再次启动 mysql 服务器

#### 修改登录密码

在 mysql 的根目录下 data 文件夹内找到 **DESKTOP-P1770NP.err** （**DESKTOP-P1770NP.err**就是系统用户名.err，每台电脑不一样）

用记事本打开 DESKTOP-P1770NP.err 找到如下内容：

```
2017-04-17T04:50:22.777490Z 1 [Note] A temporary password is generated for root@localhost: &Ga/k>jYy2Z1
```

`&Ga/k>jYy2Z1` 就是 mysql 在安装时，随机生成的 root 账号的密码。

打开 cmd.exe 输入 `mysql -u root -p`。然后，输入密码 &Ga/k>jYy2Z1，登陆成功。
在 mysql 操作界面，输入如下命令修改密码：`mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';` 
修改成功，new_password 就是新密码。

修改密码的命令还可以是：`set password for root@localhost = password('new_password');`

## mysql 8.0

navicat 连接 MySQL 报错 Authentication plugin 'caching_sha2_password' cannot be loaded

解决方案1

适用场景

    第一次构建容器/安装
    
    已安装完成后新增用户

配置

配置 mysql.cnf 配置默认身份验证插件

[mysqld]
default_authentication_plugin = mysql_native_password

解决方案2

适用场景

MySQL 已成功安装完成后

查看身份验证类型

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



root 用户的验证器插件为 caching_sha2_password
修改身份验证类型(修改密码)

mysql> ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
Query OK, 0 rows affected (0.00 sec)

mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';
Query OK, 0 rows affected (0.01 sec)


使生效

mysql> FLUSH PRIVILEGES;


验证是否生效

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


解决方案3


// 登录Mysql(需要输入密码)
mysql -u root -p

// 选择数据库(这一步不可省略)
use mysql

// 查看plugin设置
select host, user, plugin from user;

// 可以看到root的plugin是caching_sha2_password，我们希望改成mysql_native_password
ALTER USER root@localhost IDENTIFIED WITH mysql_native_password BY 'xxxxx';

// 大功告成
