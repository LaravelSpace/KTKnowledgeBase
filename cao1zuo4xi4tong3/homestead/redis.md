## 在Homestead中使用Redis

### 修改Redis密码

在命令行切换到Homestead安装目录，通过`vagrant ssh`命令连接虚拟主机 。

找到Redis配置文件`sudo find / -name redis.conf`。

更改Redis配置文件的权限`sudo chmod 755 /etc/redis/redis.conf`。

修改Redis配置文件`sudo vim /etc/redis/redis.conf`。找到requirepass参数，修改成`requirepass password`，password就是密码。如果需要外部访问还需要注释掉`bind 127.0.0.1`。修改完成后保存。

重启`Redissudo /etc/init.d/redis-server restart`。

---

| 参考来源                                                     |
| ------------------------------------------------------------ |
| [在 Homestead 中怎么配置 Redis ?](https://learnku.com/articles/9760/how-to-configure-redis-in-homestead) |