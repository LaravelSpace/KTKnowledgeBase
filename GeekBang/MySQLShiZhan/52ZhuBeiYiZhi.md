```json
{
  "updated_by": "KelipuTe",
  "updated_at": "2020-08-14",
  "tags": "极客时间,GeekBang,MySQL实战45讲,MySQL"
}
```

---

## 主备一致

MySQL能够成为现下最流行的开源数据库，binlog功不可没。binlog可以用来归档，也可以用来做主备同步。在最开始，MySQL是以容易学习和方便的高可用架构，被开发人员青睐的。而它的几乎所有的高可用架构，都直接依赖于binlog。虽然这些高可用架构已经呈现出越来越复杂的趋势，但都是从最基本的一主一备演化过来的。

### MySQL 主备的基本原理

如图1所示就是基本的主备切换流程。

![](E:\GongZuoQu\KTZhiShiKu\Image\GeekBang\MySQLShiZhan\ZhuBeiYiZhi_img02.png)

在状态1中，客户端的读写都直接访问节点A，而节点B是A的备库，只是将A的更新都同步过来，到本地执行。这样可以保持节点B和A的数据是相同的。当需要切换的时候，就切成状态2。这时候客户端读写访问的都是节点B，而节点A是B的备库。

在状态1中，虽然节点B没有被直接访问，但是我依然建议你把节点B（也就是备库）设置成只读（readonly）模式。这样做，有以下几个考虑：

- 有时候一些运营类的查询语句会被放到备库上去查，设置为只读可以防止误操作。
- 防止切换逻辑有bug，比如切换过程中出现双写，造成主备不一致。
- 可以用readonly状态，来判断节点的角色。

注意，readonly设置对超级(super)权限用户是无效的，而用于同步更新的线程，就拥有超级权限。

接下来，我们再看看节点A到B这条线的内部流程是什么样的。图2中画出的就是一个update语句在节点A执行，然后同步到节点B的完整流程图。

![](E:\GongZuoQu\KTZhiShiKu\Image\GeekBang\MySQLShiZhan\ZhuBeiYiZhi_img04.png)

图中，包含了binlog和redo log的写入机制相关的内容，可以看到：主库接收到客户端的更新请求后，执行内部事务的更新逻辑，同时写binlog。备库B跟主库A之间维持了一个长连接。主库A内部有一个线程，专门用于服务备库B的这个长连接。一个事务日志同步的完整过程是这样的：

- 在备库B上通过change master命令，设置主库A的IP、端口、用户名、密码，以及要从哪个位置开始请求binlog，这个位置包含文件名和日志偏移量。
- 在备库B上执行start slave命令，这时候备库会启动两个线程，就是图中的io_thread和sql_thread。其中io_thread负责与主库建立连接。
- 主库A校验完用户名、密码后，开始按照备库B传过来的位置，从本地读取binlog，发给B。
- 备库B拿到binlog后，写到本地文件，称为中转日志（relay log）。
- sql_thread读取中转日志，解析出日志里的命令，并执行。

这里需要说明，后来由于多线程复制方案的引入，sql_thread演化成为了多个线程，暂且不展开。

### binlog的三种格式对比

之前提到过binlog有两种格式，一种是statement，一种是row。可能在其他资料上还会看到有第三种格式，叫作mixed，其实，它就是前两种格式的混合。为了便于描述binlog的这三种格式间的区别，这里创建了一个表，并初始化几行数据。

```mysql
CREATE TABLE `t` (
  `id` int(11) NOT NULL,
  `a` int(11) DEFAULT NULL,
  `t_modified` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  KEY `a` (`a`),
  KEY `t_modified`(`t_modified`)
) ENGINE=InnoDB;

insert into t values(1,1,'2018-11-13');
insert into t values(2,2,'2018-11-12');
insert into t values(3,3,'2018-11-11');
insert into t values(4,4,'2018-11-10');
insert into t values(5,5,'2018-11-09');
```

我们来看看，如果要在表中删除一行数据的话，这个delete语句的binlog是怎么记录的。注意，下面这个语句包含注释，如果用MySQL客户端来做这个实验的话，要记得加-c参数，否则客户端会自动去掉注释。

```mysql
delete from t /*comment*/ where a>=4 and t_modified<='2018-11-10' limit 1;
```

当binlog_format=statement时，binlog里面记录的就是SQL语句的原文。你可以用`show binlog events in 'master.000001';`命令看binlog中的内容。

![](E:\GongZuoQu\KTZhiShiKu\Image\GeekBang\MySQLShiZhan\ZhuBeiYiZhi_img06.png)

现在，我们来看一下图输出结果。

- 第一行"SET @@SESSION.GTID_NEXT='ANONYMOUS’"可以先忽略。
- 第二行是一个BEGIN，跟第四行的commit对应，表示中间是一个事务；
- 第三行就是真实执行的语句了。可以看到，在真实执行的delete命令之前，还有一个"use 'test'"命令。这条命令不是我们主动执行的，而是MySQL根据当前要操作的表所在的数据库，自行添加的。这样做可以保证日志传到备库去执行的时候，不论当前的工作线程在哪个库里，都能够正确地更新到test库的表t。"use 'test'"命令之后的delete语句，就是我们输入的SQL原文了。可以看到，binlog忠实地记录了SQL命令，甚至连注释也一并记录了。
- 最后一行是一个COMMIT。你可以看到里面写着xid=61。

为了说明statement和row格式的区别，我们来看一下这条delete命令的执行效果图：

![](E:\GongZuoQu\KTZhiShiKu\Image\GeekBang\MySQLShiZhan\ZhuBeiYiZhi_img08.png)

可以看到，运行这条delete命令产生了一个warning，原因是当前binlog设置的是statement格式，并且语句中有limit，所以这个命令可能是unsafe的。这是因为delete带limit，很可能会出现主备数据不一致的情况。比如上面这个例子：

- 如果delete语句使用的是索引a，那么会根据索引a找到第一个满足条件的行，也就是说删除的是a=4这一行。
- 但如果使用的是索引t_modified，那么删除的就是t_modified='2018-11-09'也就是a=5这一行。

由于statement格式下，记录到binlog里的是语句原文，因此可能会出现这样一种情况：在主库执行这条SQL语句的时候，用的是索引a，而在备库执行这条SQL语句的时候，却使用了索引t_modified。因此，MySQL认为这样写是有风险的。那么，我们把binlog的格式改为binlog_format='row'，再看看这时候binog中的内容。

![](E:\GongZuoQu\KTZhiShiKu\Image\GeekBang\MySQLShiZhan\ZhuBeiYiZhi_img10.png)

可以看到，与statement格式的binlog相比，前后的BEGIN和COMMIT是一样的。但是，row格式的binlog里没有了SQL语句的原文，而是替换成了两个event：Table_map和Delete_rows。

- Table_map event，用于说明接下来要操作的表是test库的表t;
- Delete_rows event，用于定义删除的行为。

其实，我们通过图是看不到详细信息的，还需要借助mysqlbinlog工具，用下面这个命令解析和查看binlog中的内容。因为图5中的信息显示，这个事务的binlog是从8900这个位置开始的，所以可以用start-position参数来指定从这个位置的日志开始解析。

```mysql
mysqlbinlog -vv data/master.000001 --start-position=8900;
```

![](E:\GongZuoQu\KTZhiShiKu\Image\GeekBang\MySQLShiZhan\ZhuBeiYiZhi_img12.png)

从这个图6中，我们可以看到以下几个信息：

- server id 1，表示这个事务是在server_id=1的这个库上执行的。
- 每个event都有CRC32的值，这是因为参数binlog_checksum设置成了CRC32。
- Table_map event跟在图中看到的相同，显示了接下来要打开的表，map到数字226。现在我们这条SQL语句只操作了一张表。如果要操作多张表每个表都有一个对应的Table_map event、都会map到一个单独的数字，用于区分对不同表的操作。
- 我们在mysqlbinlog的命令中，使用了-vv参数是为了把内容都解析出来，所以从结果里面可以看到各个字段的值（比如，@1=4、@2=4这些值）。
- binlog_row_image的默认配置是FULL，因此Delete_event里面，包含了删掉的行的所有字段的值。如果把binlog_row_image设置为MINIMAL，则只会记录必要的信息，在这个例子里，就是只会记录id=4这个信息。
- 最后的Xid event，用于表示事务被正确地提交了。

可以看到，当binlog_format使用row格式的时候，binlog里面记录了真实删除行的主键id，这样binlog传到备库去的时候，就肯定会删除id=4的行，不会有主备删除不同行的问题。

### mixed格式的binlog

- 因为有些statement格式的binlog可能会导致主备不一致，所以要使用row格式。
- 但row格式的缺点是，很占空间。比如你用一个delete语句删掉10万行数据，用statement的话就是一个SQL语句被记录到binlog中，占用几十个字节的空间。但如果用row格式的binlog，就要把这10万条记录都写到binlog中。这样做，不仅会占用更大的空间，同时写binlog也要耗费IO资源，影响执行速度。
- 所以，MySQL就取了个折中方案，也就是有了mixed格式的binlog。mixed格式的意思是，MySQL自己会判断这条SQL语句是否可能引起主备不一致，如果有可能，就用row格式，否则就用statement格式。

也就是说，mixed格式可以利用statment格式的优点，同时又避免了数据不一致的风险。因此，如果线上MySQL设置的binlog格式是statement的话，那基本上就可以认为这是一个不合理的设置。你至少应该把binlog的格式设置为mixed。比如我们这个例子，设置为mixed后，就会记录为row格式。而如果执行的语句去掉limit 1，就会记录为statement格式。

现在越来越多的场景要求把MySQL的binlog格式设置成row。这么做的理由有很多，我来给你举一个可以直接看出来的好处：恢复数据。我们分别从delete、insert和update这三种SQL语句的角度，来看看数据恢复的问题。

通过上文可以看出，即使执行的是delete语句，row格式的binlog也会把被删掉的行的整行信息保存起来。所以，如果在执行完一条delete语句以后，发现删错数据了，可以直接把binlog中记录的delete语句转成insert，把被错删的数据插入回去就可以恢复了。

如果执行错了insert语句就更直接了。row格式下，insert语句的binlog里会记录所有的字段信息，这些信息可以用来精确定位刚刚被插入的那一行。这时，直接把insert语句转成delete语句，删除掉这被误插入的一行数据就可以了。

如果执行的是update语句的话，binlog里面会记录修改前整行的数据和修改后的整行数据。所以，如果你误执行了update语句的话，只需要把这个event前后的两行信息对调一下，再去数据库里面执行，就能恢复这个更新操作了。

其实，由delete、insert或者update语句导致的数据操作错误，需要恢复到操作之前状态的情况，也时有发生。MariaDB的Flashback工具就是基于上面介绍的原理来回滚数据的。

虽然mixed格式的binlog现在已经用得不多了，但这里还是要再借用一下mixed格式来说明一个问题，看一下这条SQL语句：`insert into t values(10,10, now());`。

如果我们把binlog格式设置为mixed，来看一下这条语句执行的效果。可以看到，MySQL 用的居然是statement格式。

![](E:\GongZuoQu\KTZhiShiKu\Image\GeekBang\MySQLShiZhan\ZhuBeiYiZhi_img14.png)

![](E:\GongZuoQu\KTZhiShiKu\Image\GeekBang\MySQLShiZhan\ZhuBeiYiZhi_img16.png)

我们再用mysqlbinlog工具来看看。从图中的结果可以看到，原来binlog在记录event的时候，多记了一条命令：SETTIMESTAMP=1546103491。它用SET TIMESTAMP命令约定了接下来的now()函数的返回时间。因此，不论这个binlog是1分钟之后被备库执行，还是3天后用来恢复这个库的备份，这个insert语句插入的行，值都是固定的。也就是说，通过这条SET TIMESTAMP命令，MySQL就确保了主备数据的一致性。

所以，在重放binlog数据的时候，直接用mysqlbinlog解析出日志，然后把里面的statement语句直接拷贝出来执行，这个方法是有风险的。因为有些语句的执行结果是依赖于上下文命令的，直接执行的结果很可能是错误的。所以，用binlog来恢复数据的标准做法是，用mysqlbinlog工具解析出来，然后把解析结果整个发给MySQL执行。类似下面的命令：

```mysql
mysqlbinlog master.000001 --start-position=2738 --stop-position=2973 | mysql -h127.0.0.1 -P13000 -u$user -p$pwd;
```

这个命令的意思是，将master.000001文件里面从第2738字节到第2973字节中间这段内容解析出来，放到MySQL去执行。

### 循环复制问题

binlog的特性确保了在备库执行相同的binlog，可以得到与主库相同的状态。因此，可以认为正常情况下主备的数据是一致的。也就是说，图1中A、B两个节点的内容是一致的。其实，图1中的是M-S结构，但实际生产上使用比较多的是双M结构，也就是图9所示的主备切换流程。

![](E:\GongZuoQu\KTZhiShiKu\Image\GeekBang\MySQLShiZhan\ZhuBeiYiZhi_img18.png)

对比图9和图1，你可以发现，双M结构和M-S结构，其实区别只是多了一条线，即：节点A和B之间总是互为主备关系。这样在切换的时候就不用再修改主备关系。

但是，双M结构还有一个问题需要解决。业务逻辑在节点A上更新了一条语句，然后再把生成的binlog发给节点B，节点B执行完这条更新语句后也会生成binlog。（建议把参数log_slave_updates设置为on，表示备库执行relay log后生成binlog）。那么，如果节点A同时是节点B的备库，相当于又把节点B新生成的binlog拿过来执行了一次，然后节点A和B间，会不断地循环执行这个更新语句，也就是循环复制了。

从上面的图6中可以看到，MySQL在binlog中记录了这个命令第一次执行时所在实例的server id。因此，可以用下面的逻辑，来解决两个节点间的循环复制的问题：

- 规定两个库的server id必须不同，如果相同，则它们之间不能设定为主备关系；
- 一个备库接到binlog并在重放的过程中，生成与原binlog的server id相同的新的binlog；
- 每个库在收到从自己的主库发过来的日志后，先判断server id，如果跟自己的相同，表示这个日志是自己生成的，就直接丢弃这个日志。

按照这个逻辑，如果我们设置了双M结构，日志的执行流就会变成这样：

- 从节点A更新的事务，binlog里面记的都是A的server id；
- 传到节点B执行一次以后，节点B生成的binlog的server id也是A的server id；
- 再传回给节点A，A判断到这个server id与自己的相同，就不会再处理这个日志。所以，死循环在这里就断掉了。

但是，这个机制其实并不完备，在某些场景下，还是有可能出现死循环。

- 一种场景是，在一个主库更新事务后，用命令set global server_id=x修改了server_id。等日志再传回来的时候，发现server_id跟自己的server_id不同，就只能执行了。
- 另一种场景是，有三个节点的时候，如图所示，trx1是在节点B执行的，因此binlog上的server_id 就是B，binlog传给节点A，然后A和A'搭建了双M结构，就会出现循环复制。

![](E:\GongZuoQu\KTZhiShiKu\Image\GeekBang\MySQLShiZhan\ZhuBeiYiZhi_img20.png)

这种三节点复制的场景，做数据库迁移的时候会出现。如果出现了循环复制，可以在A或者A'上，执行如下命令：

```mysql
stop slave;
CHANGE MASTER TO IGNORE_SERVER_IDS=(server_id_of_B);
start slave;
```

这样这个节点收到日志后就不会再执行。过一段时间后，再执行下面的命令把这个值改回来。

```mysql
stop slave;
CHANGE MASTER TO IGNORE_SERVER_IDS=();
start slave;
```


