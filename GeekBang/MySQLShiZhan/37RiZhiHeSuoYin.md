```json
{
  "updated_by": "KelipuTe",
  "updated_at": "2020-07-05",
  "tags": "极客时间,GeekBang,MySQL实战45讲,MySQL"
}
```

---

## 日志和索引相关问题

### 日志相关问题

在两阶段提交的不同瞬间，MySQL 如果发生异常重启，是怎么保证数据完整性的？

![](E:\GongZuoQu\KTZhiShiKu\Image\GeekBang\MySQLShiZhan\RiZhiHeSuoYin_img02.jpg)

图中的这个commit步骤，指的是事务提交过程中的一个步骤，也是最后一步。当这个步骤执行完成后，这个事务就提交完成了。这里注意和commit语句区分，commit语句执行的时候，会包含commit步骤。这个更新数据的例子里面，没有显式地开启事务，因此这个update语句自己就是一个事务，在执行完成后提交事务时，就会用到这个commit步骤。

如果在图中时刻A的地方，也就是写入redo log处于prepare阶段之后、写binlog之前，发生了崩溃（crash），由于此时binlog还没写，redo log也还没提交，所以崩溃恢复的时候，这个事务会回滚。这时候，binlog还没写，所以也不会传到备库。

如果在图中时刻B的地方，也就是binlog写完，redo log还没commit前发生crash，那崩溃恢复的时候MySQL会怎么处理？我们先来看一下崩溃恢复时的判断规则。

- 1、如果redo log里面的事务是完整的，也就是已经有了commit标识，则直接提交；

- 2、如果redo log里面的事务只有完整的prepare，则判断对应的事务binlog是否存在并完整：
  - a.如果是，则提交事务；
  - b.否则，回滚事务。

这里，时刻B发生crash对应的就是2(a)的情况，崩溃恢复过程中事务会被提交。

#### 追问1：MySQL怎么知道binlog是完整的

回答：一个事务的binlog是有完整格式的：statement格式的binlog，最后会有COMMIT；row格式的binlog，最后会有一个XID event。

另外，在MySQL5.6.2版本以后，还引入了binlog-checksum参数，用来验证binlog内容的正确性。对于binlog日志由于磁盘原因，可能会在日志中间出错的情况，MySQL可以通过校验checksum的结果来发现。所以，MySQL还是有办法验证事务binlog的完整性的。

#### 追问2：redolog和binlog是怎么关联起来的

回答：它们有一个共同的数据字段，叫XID。崩溃恢复的时候，会按顺序扫描redo log：如果碰到既有prepare、又有commit的redo log，就直接提交；如果碰到只有parepare、而没有commit的redolog，就拿着XID去binlog找对应的事务。

#### 追问3：处于prepare阶段的redo log加上完整binlog，重启就能恢复，MySQL为什么要这么设计

回答：其实，这个问题还是跟我们在反证法中说到的数据与备份的一致性有关。在时刻B，也就是binlog写完以后MySQL发生崩溃，这时候binlog已经写入了，之后就会被从库（或者用这个binlog恢复出来的库）使用。所以，在主库上也要提交这个事务。采用这个策略，主库和备库的数据就保证了一致性。

#### 追问4：如果这样的话，为什么还要两阶段提交呢。干脆先redo log写完，再写binlog。崩溃恢复的时候，必须得两个日志都完整才可以。是不是一样的逻辑

回答：其实，两阶段提交是经典的分布式系统问题，并不是MySQL独有的。如果必须要举一个场景，来说明这么做的必要性的话，那就是事务的持久性问题。对于InnoDB引擎来说，如果redo log提交完成了，事务就不能回滚（如果这还允许回滚，就可能覆盖掉别的事务的更新）。而如果redo log直接提交，然后binlog写入的时候失败，InnoDB又回滚不了，数据和binlog日志又不一致了。两阶段提交就是为了给所有人一个机会，当每个人都说我ok的时候，再一起提交。

#### 追问5：不引入两个日志，也就没有两阶段提交的必要了。只用binlog来支持崩溃恢复，又能支持归档，不就可以了

答案是不可以。如果说历史原因的话，那就是InnoDB并不是MySQL的原生存储引擎。MySQL的原生引擎是MyISAM，设计之初就有没有支持崩溃恢复。

而如果说实现上的原因的话，就有很多了。就按照问题中说的，只用binlog来实现崩溃恢复的流程，我画了一张示意图，这里就没有redo log了。这样的流程下，binlog还是不能支持崩溃恢复的。我说一个不支持的点吧：binlog没有能力恢复数据页。如果在图中标的位置，也就是binlog2写完了，但是整个事务还没有commit的时候，MySQL发生了crash。重启后，引擎内部事务2会回滚，然后应用binlog2可以补回来；但是对于事务1来说，系统已经认为提交完成了，不会再应用一次binlog1。

![](E:\GongZuoQu\KTZhiShiKu\Image\GeekBang\MySQLShiZhan\RiZhiHeSuoYin_img04.jpg)

InnoDB在作为MySQL的插件加入MySQL引擎家族之前，就已经是一个提供了崩溃恢复和事务支持的引擎了。InnoDB接入了MySQL后，发现既然binlog没有崩溃恢复的能力，那就用InnoDB原有的redo log好了。InnoDB引擎使用的是WAL技术，执行事务的时候，写完内存和日志，事务就算完成了。如果之后崩溃，要依赖于日志来恢复数据页。

也就是说在图中这个位置发生崩溃的话，事务1也是可能丢失了的，而且是数据页级的丢失。此时，binlog里面并没有记录数据页的更新细节，是补不回来的。你如果要说，那我优化一下binlog的内容，让它来记录数据页的更改可以吗？但，这其实就是又做了一个redo log出来。所以，至少现在的binlog能力，还不能支持崩溃恢复。

#### 追问6：那能不能反过来，只用redo log，不要binlog？

回答：如果只从崩溃恢复的角度来讲是可以的。你可以把binlog关掉，这样就没有两阶段提交了，但系统依然是crash-safe的。

但是，如果你了解一下业界各个公司的使用场景的话，就会发现在正式的生产库上，binlog都是开着的。因为binlog有着redo log无法替代的功能。一个是归档。redo log是循环写，写到末尾是要回到开头继续写的。这样历史日志没法保留，redo log也就起不到归档的作用。一个就是MySQL系统依赖于binlog。binlog作为MySQL一开始就有的功能，被用在了很多地方。其中，MySQL系统高可用的基础，就是binlog复制。

还有很多公司有异构系统（比如一些数据分析系统），这些系统就靠消费MySQL的binlog来更新自己的数据。关掉binlog的话，这些下游系统就没法输入了。总之，由于现在包括MySQL高可用在内的很多系统机制都依赖于binlog。

#### 追问7：redo log一般设置多大

回答：redo log太小的话，会导致很快就被写满，然后不得不强行刷redo log，这样WAL机制的能力就发挥不出来了。所以，如果是现在常见的几个TB的磁盘的话，就不要太小气了，直接将redo log设置为4个文件、每个文件1GB吧。

#### 追问8：正常运行中的实例，数据写入后的最终落盘，是从redo log更新过来的还是从buffer pool更新过来的呢

实际上，redo log并没有记录数据页的完整数据，所以它并没有能力自己去更新磁盘数据页，也就不存在数据最终落盘，是由redo log更新过去的情况。

- 1、如果是正常运行的实例的话，数据页被修改以后，跟磁盘的数据页不一致，称为脏页。最终数据落盘，就是把内存中的数据页写盘。这个过程，甚至与redo log毫无关系。
- 2、在崩溃恢复场景中，InnoDB如果判断到一个数据页可能在崩溃恢复的时候丢失了更新，就会将它读到内存，然后让redo log更新内存内容。更新完成后，内存页变成脏页，就回到了第一种情况的状态。

#### 追问9：redo log buffer是什么？是先修改内存，还是先写redo log文件

在一个事务的更新过程中，日志是要写多次的。比如下面这个事务：

```mysql
begin;
insert into t1 ...
insert into t2 ...
commit;
```

这个事务要往两个表中插入记录，插入数据的过程中，生成的日志都得先保存起来，但又不能在还没commit的时候就直接写到redo log文件里。所以，redo log buffer就是一块内存，用来先存redo日志的。也就是说，在执行第一个insert的时候，数据的内存被修改了，redo log buffer也写入了日志。但是，真正把日志写到redo log文件（文件名是ib_logfile+数字），是在执行commit语句的时候做的。

这里说的是事务执行过程中不会主动去刷盘，以减少不必要的IO消耗。但是可能会出现被动写入磁盘，比如内存不够、其他事务提交等情况。

单独执行一个更新语句的时候，InnoDB会自己启动一个事务，在语句执行完成的时候提交。过程跟上面是一样的，只不过是压缩到了一个语句里面完成。

### 索引设计问题

业务上有这样的需求，A、B两个用户，如果互相关注，则成为好友。设计上是有两张表，一个是like表，一个是friend表，like表有user_id、liker_id两个字段，我设置为复合唯一索引即uk_user_id_liker_id。语句执行逻辑是这样的：

以A关注B为例：第一步，先查询对方有没有关注自己（B有没有关注A） `select * from like where user_id = B and liker_id = A;` 如果有，则成为好友`insert into friend;`。没有，则只是单向关注关系`insert into like;`。

但是如果A、B同时关注对方，会出现不会成为好友的情况。因为上面第1步，双方都没关注对方。第1步即使使用了排他锁也不行，因为记录不存在，行锁无法生效。请问这种情况，在MySQL锁层面有没有办法处理？

```mysql
CREATE TABLE `like` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `user_id` int(11) NOT NULL,
  `liker_id` int(11) NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_user_id_liker_id` (`user_id`,`liker_id`)
) ENGINE=InnoDB;

CREATE TABLE `friend` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `friend_1_id` int(11) NOT NULL,
  `friend_2_id` int(11) NOT NULL,
  UNIQUE KEY `uk_friend` (`friend_1_id`,`friend_2_id`),
  PRIMARY KEY (`id`)
) ENGINE=InnoDB;
```

在并发场景下，同时有两个人，设置为关注对方，就可能导致无法成功加为朋友关系。用时刻顺序表的形式，把这两个事务的执行语句列出来：

| session1(A like B)                                     | session2(B like A)                                     |
| ------------------------------------------------------ | ------------------------------------------------------ |
| begin;                                                 |                                                        |
| select * from like where user_id = B and liker_id = A; |                                                        |
|                                                        | begin;                                                 |
|                                                        | select * from like where user_id = A and liker_id = B; |
|                                                        | insert into like (user_id,liker_id) values (B,A);      |
| insert into like (user_id,liker_id) values (A,B);      |                                                        |
| commit;                                                |                                                        |
|                                                        | commit;                                                |

由于一开始A和B之间没有关注关系，所以两个事务里面的select语句查出来的结果都是空。因此，session1的逻辑就是既然B没有关注A，那就只插入一个单向关注关系。session2也同样是这个逻辑。这个结果对业务来说就是bug了。因为在业务设定里面，这两个逻辑都执行完成以后，是应该在friend表里面插入一行记录的。

如提问里面说的，第1步即使使用了排他锁也不行，因为记录不存在，行锁无法生效。不过，有另外一个方法，来解决这个问题。首先，要给like表增加一个字段，比如叫作relation_ship，并设为整型，取值1、2、3。值是1的时候，表示user_id关注liker_id；值是2的时候，表示liker_id关注user_id；值是3的时候，表示互相关注。

然后，当A关注B的时候，逻辑改成如下所示的样子：应用代码里面，比较A和B的大小，如果A<B，就执行下面的逻辑：

```mysql
mysql> begin; /*启动事务*/
insert into `like`(user_id, liker_id, relation_ship) values(A, B, 1) on duplicate key update relation_ship=relation_ship | 1;
select relation_ship from `like` where user_id=A and liker_id=B;
/*代码中判断返回的 relation_ship，
  如果是1，事务结束，执行 commit
  如果是3，则执行下面这两个语句：
  */
insert ignore into friend(friend_1_id, friend_2_id) values(A,B);
commit;
```

如果A>B，则执行下面的逻辑：

```mysql
mysql> begin; /*启动事务*/
insert into `like`(user_id, liker_id, relation_ship) values(B, A, 2) on duplicate key update relation_ship=relation_ship | 2;
select relation_ship from `like` where user_id=B and liker_id=A;
/*代码中判断返回的 relation_ship，
  如果是2，事务结束，执行 commit
  如果是3，则执行下面这两个语句：
*/
insert ignore into friend(friend_1_id, friend_2_id) values(B,A);
commit;
```

这个设计里，让like表里的数据保证user_id<liker_id，这样不论是A关注B，还是B关注A，在操作like表的时候，如果反向的关系已经存在，就会出现行锁冲突。然后，`insert … on duplicate`语句，确保了在事务内部，执行了这个SQL语句后，就强行占住了这个行锁，之后的select判断relation_ship这个逻辑时就确保了是在行锁保护下的读操作。操作符“|”是按位或，连同最后一句insert语句里的ignore，是为了保证重复调用时的幂等性。

这样，即使在双方同时执行关注操作，最终数据库里的结果，也是like表里面有一条关于A和B的记录，而且relation_ship的值是3，并且friend表里面也有了A和B的这条记录。

这里要注意，在业务开发保证不会插入重复记录的情况下，着重要解决性能问题的时候，才建议尽量使用普通索引。而像这个例子里，业务根本就是保证我一定会插入重复数据，数据库一定要要有唯一性约束，这时唯一索引就没啥好说的了。

### 思考题

我们创建了一个简单的表t，并插入一行，然后对这一行做修改。

```mysql
mysql> CREATE TABLE `t` (
  `id` int(11) NOT NULL primary key auto_increment,
  `a` int(11) DEFAULT NULL
) ENGINE=InnoDB;
insert into t values(1,2);
```

这时候，表t里有唯一的一行数据 (1,2)。假设，我现在要执行：

```mysql
mysql> update t set a=2 where id=1;
```

你会看到这样的结果：

![](E:\GongZuoQu\KTZhiShiKu\Image\GeekBang\MySQLShiZhan\RiZhiHeSuoYin_img06.png)

结果显示，匹配(rows matched)了一行，修改(Changed)了0行。仅从现象上看，MySQL内部在处理这个命令的时候，可以有以下三种选择：

- 1、更新都是先读后写的，MySQL读出数据，发现a的值本来就是2，不更新，直接返回，执行结束；
- 2、MySQL调用了InnoDB引擎提供的修改为(1,2)这个接口，但是引擎发现值与原来相同，不更新，直接返回；
- 3、InnoDB认真执行了把这个值修改成(1,2)这个操作，该加锁的加锁，该更新的更新。

你觉得实际情况会是以上哪种呢？你可否用构造实验的方式，来证明你的结论？进一步地，可以思考一下，MySQL为什么要选择这种策略呢？

第一个选项是，MySQL读出数据，发现值与原来相同，不更新，直接返回，执行结束。这里我们可以用一个锁实验来确认。假设当前表t里的值是(1,2)。sessionB的update语句被blocked了，加锁这个动作是InnoDB才能做的，所以排除选项1。

| sessionA                     | sessionB                              |
| ---------------------------- | ------------------------------------- |
| brgin;                       |                                       |
| update t set a=2 where id=1; |                                       |
|                              | update t set a=2 where id=1;(blocked) |

第二个选项是，MySQL调用了InnoDB引擎提供的接口，但是引擎发现值与原来相同，不更新，直接返回。有没有这种可能呢？这里我用一个可见性实验来确认。假设当前表里的值是(1,2)。sessionA的第二个select语句是一致性读（快照读)，它是不能看见sessionB的更新的。现在它返回的是(1,3)，表示它看见了某个新的版本，这个版本只能是sessionA自己的update语句做更新的时候生成。

| sessionA                                                     | sessionB                     |
| ------------------------------------------------------------ | ---------------------------- |
| brgin;                                                       |                              |
| select * from t where id=1;                                  |                              |
| 返回(1,2)                                                    |                              |
|                                                              | update t set a=3 where id=1; |
| update t set a=3 where id=1;                                 |                              |
| Query OK,0 row affected (0.00 sec)<br>Rows matched:1 Changed:0 Warnings:0 |                              |
| select * from t where id=1;                                  |                              |
| 返回(1,3)                                                    |                              |

答案应该是选项3，即：InnoDB认真执行了把这个值修改成(1,2)这个操作，该加锁的加锁，该更新的更新。那么如果MySQL判断一下，是不是就不用浪费InnoDB操作，多去更新一次了？其实MySQL是确认了的。只是在这个语句里面，MySQL认为读出来的值，只有一个确定的(id=1)，而要写的是(a=3)，只从这两个信息是看不出来不需要修改的。作为验证，你可以看一下下面这个例子。

| sessionA                                                     | sessionB                     |
| ------------------------------------------------------------ | ---------------------------- |
| brgin;                                                       |                              |
| select * from t where id=1;                                  |                              |
| 返回(1,2)                                                    |                              |
|                                                              | update t set a=3 where id=1; |
| update t set a=3 where id=1 and a=3;                         |                              |
| Query OK,0 row affected (0.00 sec)<br>Rows matched:1 Changed:0 Warnings:0 |                              |
| select * from t where id=1;                                  |                              |
| 返回(1,2)                                                    |                              |

### 为什么要学原理

有的新人会问为什么需要这么麻烦，我执行一下，看看结果对不对，对了就行，不对就改，是不是也可以？不可以。因为如果这样，我们就会受到很多局限，即使我们定位自己是业务开发人员。

这会限制基于数据库的业务架构能力。一个语句可以试，一个五个语句的事务分析就要试很多次，一个复杂业务系统的数据库设计，是试不出来的。原理可以帮我们剪枝，排除掉那些理论上明显错误的方案，这样才有精力真的去试那些有限的、可能正确的方案。

我们不需要100%精通MySQL，但是只要多知道一些原理，就能多剪一些枝，架构设计就能少一些错误选项的干扰，设计出来的项目架构正确的可能性更高。

### 其他问题

问题1：执行一个update语句以后，我再去执行hexdump命令直接查看ibd文件内容，为什么没有看到数据有改变呢？回答：这可能是因为WAL机制的原因。update语句执行完成后，InnoDB只保证写完了redolog、内存，可能还没来得及将数据写到磁盘。

问题2：为什么binlog cache是每个线程自己维护的，而redo log buffer是全局共用的？回答：MySQL这么设计的主要原因是，binlog是不能被打断的。一个事务的binlog必须连续写，因此要整个事务完成后，再一起写到文件里。而redo log并没有这个要求，中间有生成的日志可以写到redo log buffer中。redo log buffer中的内容还能搭便车，其他事务提交的时候可以被一起写到磁盘中。

问题3：事务执行期间，还没到提交阶段，如果发生crash的话，redo log肯定丢了，这会不会导致主备不一致呢？回答：不会。因为这时候binlog也还在binlog cache里，没发给备库。crash以后redo log和binlog都没有了，从业务角度看这个事务也没有提交，所以数据是一致的。

问题4：如果binlog写完盘以后发生crash，这时候还没给客户端答复就重启了。等客户端再重连进来，发现事务已经提交成功了，这是不是bug？回答：不是。你可以设想一下更极端的情况，整个事务都提交成功了，redo log commit完成了，备库也收到binlog并执行了。但是主库和客户端网络断开了，导致事务成功的包返回不回去，这时候客户端也会收到网络断开的异常。这种也只能算是事务成功的，不能认为是bug。

实际上数据库的crash-safe保证的是：

- 如果客户端收到事务成功的消息，事务就一定持久化了；
- 如果客户端收到事务失败（比如主键冲突、回滚等）的消息，事务就一定失败了；
- 如果客户端收到执行异常的消息，应用需要重连后通过查询当前状态来继续后续的逻辑。此时数据库只需要保证内部（数据和日志之间，主库和备库之间）一致就可以了。