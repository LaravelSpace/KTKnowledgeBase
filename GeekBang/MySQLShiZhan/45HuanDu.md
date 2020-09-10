```json
{
  "updated_by": "KelipuTe",
  "updated_at": "2020-07-06",
  "tags": "极客时间,GeekBang,MySQL实战45讲,MySQL"
}
```

---

## 幻读

为了便于说明问题，我们就先使用一个小一点儿的表。建表和初始化语句如下：

```mysql
CREATE TABLE `t` (
  `id` int(11) NOT NULL,
  `c` int(11) DEFAULT NULL,
  `d` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `c` (`c`)
) ENGINE=InnoDB;
insert into t values(0,0,0),(5,5,5),(10,10,10),(15,15,15),(20,20,20),(25,25,25);
```

这个表除了主键id外，还有一个索引c，初始化语句在表中插入了6行数据。44ZhiChaYiTiao.md的问题是，下面的语句序列，是怎么加锁的，加的锁又是什么时候释放的呢？

```mysql
begin;
select * from t where d=5 for update;
commit;
```

比较好理解的是，这个语句会命中d=5的这一行，对应的主键id=5，因此在select语句执行完成后，id=5这一行会加一个写锁，而且由于两阶段锁协议，这个写锁会在执行commit语句的时候释放。

由于字段d上没有索引，因此这条查询语句会做全表扫描。那么，其他被扫描到的，但是不满足条件的5行记录上，会不会被加锁呢？我们知道，InnoDB的默认事务隔离级别是可重复读，所以本文接下来没有特殊说明的部分，都是设定在可重复读隔离级别下。

### 幻读是什么

现在，我们就来分析一下，如果只在 id=5 这一行加锁，而其他行的不加锁的话，会怎么样。下面先来看一下这个场景（注意：这是假设的一个场景）：

|      | sessionA                                                     | sessionB                     | sessionC                       |
| ---- | ------------------------------------------------------------ | ---------------------------- | ------------------------------ |
| T1   | begin;                                                       |                              |                                |
|      | select * from t where d=5 for update;<br>/* Q1 */ result:(5,5,5) |                              |                                |
| T2   |                                                              | update t set d=5 where id=0; |                                |
| T3   | select * from t where d=5 for update;<br>/* Q2 */ result:(0,0,5),(5,5,5) |                              |                                |
| T4   |                                                              |                              | insert into t values  (1,1,5); |
| T5   | select * from t where d=5 for update;<br>/* Q3 */ result:(0,0,5),(1,1,5),(5,5,5) |                              |                                |
| T6   | commit;                                                      |                              |                                |

可以看到，sessionA里执行了三次查询，分别是Q1、Q2和Q3。它们的SQL语句相同，都是select * from t where d=5 for update。这个语句的意思你应该很清楚了，查所有d=5的行，而且使用的是当前读，并且加上写锁。现在，我们来看一下这三条SQL语句，分别会返回什么结果。

- Q1只返回id=5这一行；
- 在T2时刻，sessionB把id=0这一行的d值改成了5，因此T3时刻Q2查出来的是id=0和id=5这两行；
- 在T4时刻，sessionC又插入一行（1,1,5），因此T5时刻Q3查出来的是id=0、id=1和id=5的这三行。

其中，Q3读到id=1这一行的现象，被称为幻读。也就是说，幻读指的是一个事务在前后两次查询同一个范围的时候，后一次查询看到了前一次查询没有看到的行。

这里，我需要对幻读做一个说明：

- 在可重复读隔离级别下，普通的查询是快照读，是不会看到别的事务插入的数据的。因此，幻读在当前读下才会出现。
- 上面sessionB的修改结果，被sessionA之后的select语句用当前读看到，不能称为幻读。幻读仅专指新插入的行。

我们学到的事务可见性规则来分析的话，上面这三条SQL语句的返回结果都没有问题。因为这三个查询都是加了for update，都是当前读。而当前读的规则，就是要能读到所有已经提交的记录的最新值。并且，sessionB和sessionC的两条语句，执行后就会提交，所以Q2和Q3就是应该看到这两个事务的操作效果，而且也看到了，这跟事务的可见性规则并不矛盾。

但是，这是不是真的没问题呢？不，这里还真就有问题。

### 幻读有什么问题

首先是语义上的。sessionA在T1时刻就声明了，我要把所有d=5的行锁住，不准别的事务进行读写操作。而实际上，这个语义被破坏了。如果现在这样看感觉还不明显的话，我再往sessionB和sessionC里面分别加一条SQL语句，你再看看会出现什么现象。

|      | sessionA                                      | sessionB                     | sessionC                       |
| ---- | --------------------------------------------- | ---------------------------- | ------------------------------ |
| T1   | begin;                                        |                              |                                |
|      | select * from t where d=5 for update;/* Q1 */ |                              |                                |
| T2   |                                               | update t set d=5 where id=0; |                                |
|      |                                               | update t set c=5 where id=0; |                                |
| T3   | select * from t where d=5 for update;/* Q2 */ |                              |                                |
| T4   |                                               |                              | insert into t values  (1,1,5); |
|      |                                               |                              | update t set c=5 where id=1;   |
| T5   | select * from t where d=5 for update;/* Q3 */ |                              |                                |
| T6   | commit;                                       |                              |                                |

sessionB的第二条语句update t set c=5 where id=0，语义是我把id=0、d=5这一行的c值，改成了5。由于在T1时刻，sessionA还只是给id=5这一行加了行锁，并没有给id=0这行加上锁。因此，sessionB在T2时刻，是可以执行这两条update语句的。这样，就破坏了sessionA里Q1语句要锁住所有d=5的行的加锁声明。sessionC也是一样的道理，对id=1这一行的修改，也是破坏了Q1的加锁声明。

### 数据一致性问题

我们知道，锁的设计是为了保证数据的一致性。而这个一致性，不止是数据库内部数据状态在此刻的一致性，还包含了数据和日志在逻辑上的一致性。为了说明这个问题，我给 sessionA在T1时刻再加一个更新语句，即：update t set d=100 where d=5。

|      | sessionA                                      | sessionB                     | sessionC                       |
| ---- | --------------------------------------------- | ---------------------------- | ------------------------------ |
| T1   | begin;                                        |                              |                                |
|      | select * from t where d=5 for update;/* Q1 */ |                              |                                |
|      | update t set d=100 where d=5;                 |                              |                                |
| T2   |                                               | update t set d=5 where id=0; |                                |
|      |                                               | update t set c=5 where id=0; |                                |
| T3   | select * from t where d=5 for update;/* Q2 */ |                              |                                |
| T4   |                                               |                              | insert into t values  (1,1,5); |
|      |                                               |                              | update t set c=5 where id=1;   |
| T5   | select * from t where d=5 for update;/* Q3 */ |                              |                                |
| T6   | commit;                                       |                              |                                |

update的加锁语义和select  … for update是一致的，所以这时候加上这条update语句也很合理。sessionA声明说要给d=5的语句加上锁，就是为了要更新数据，新加的这条update语句就是把它认为加上了锁的这一行的d值修改成了100。现在，我们来分析一下表3执行完成后，数据库里会是什么结果。

- 经过T1时刻，id=5这一行变成(5,5,100)，当然这个结果最终是在T6时刻正式提交的;
- 经过T2时刻，id=0这一行变成(0,5,5);
- 经过T4时刻，表里面多了一行(1,5,5);
- 其他行跟这个执行序列无关，保持不变。

这样看，这些数据也没啥问题，但是我们再来看看这时候binlog里面的内容。

- T2时刻，sessionB事务提交，写入了两条语句；
- T4时刻，sessionC事务提交，写入了两条语句；
- T6时刻，sessionA事务提交，写入了update t set d=100 where d=5这条语句。

我统一放到一起的话，就是这样的：

```mysql
update t set d=5 where id=0; /* (0,0,5) */
update t set c=5 where id=0; /* (0,5,5) */

insert into t values(1,1,5); /* (1,1,5) */
update t set c=5 where id=1; /* (1,5,5) */

update t set d=100 where d=5; /* 所有d=5的行，d改成100 */
```

好，你应该看出问题了。这个语句序列，不论是拿到备库去执行，还是以后用binlog来克隆一个库，这三行的结果，都变成了(0,5,100)、(1,5,100)和(5,5,100)。也就是说，id=0和id=1这两行，发生了数据不一致。这个问题很严重，是不行的。到这里，我们再回顾一下，这个数据不一致到底是怎么引入的？

我们分析一下可以知道，这是我们假设select * from t where d=5 for update这条语句只给d=5这一行，也就是id=5的这一行加锁导致的。所以我们认为，上面的设定不合理，要改。那怎么改呢？我们把扫描过程中碰到的行，也都加上写锁，再来看看执行效果。表4假设扫描到的行都被加上了行锁。

|      | sessionA                                      | sessionB                              | sessionC                       |
| ---- | --------------------------------------------- | ------------------------------------- | ------------------------------ |
| T1   | begin;                                        |                                       |                                |
|      | select * from t where d=5 for update;/* Q1 */ |                                       |                                |
|      | update t set d=100 where d=5;                 |                                       |                                |
| T2   |                                               | update t set d=5 where id=0;(blocked) |                                |
|      |                                               | update t set c=5 where id=0;          |                                |
| T3   | select * from t where d=5 for update;/* Q2 */ |                                       |                                |
| T4   |                                               |                                       | insert into t values  (1,1,5); |
|      |                                               |                                       | update t set c=5 where id=1;   |
| T5   | select * from t where d=5 for update;/* Q3 */ |                                       |                                |
| T6   | commit;                                       |                                       |                                |

```mysql
insert into t values(1,1,5); /*(1,1,5)*/
update t set c=5 where id=1; /*(1,5,5)*/

update t set d=100 where d=5; /*所有d=5的行，d改成100*/

update t set d=5 where id=0; /*(0,0,5)*/
update t set c=5 where id=0; /*(0,5,5)*/
```

可以看到，按照日志顺序执行，id=0这一行的最终结果也是(0,5,5)。所以，id=0这一行的问题解决了。但同时你也可以看到，id=1这一行，在数据库里面的结果是(1,5,5)，而根据binlog的执行结果是(1,5,100)，也就是说幻读的问题还是没有解决。为什么我们已经这么凶残地，把所有的记录都上了锁，还是阻止不了id=1这一行的插入和更新呢？

原因很简单。在T3时刻，我们给所有行加锁的时候，id=1这一行还不存在，不存在也就加不上锁。也就是说，即使把所有的记录都加上锁，还是阻止不了新插入的记录，这也是为什么幻读会被单独拿出来解决的原因。到这里，其实我们刚说明完文章的标题：幻读的定义和幻读有什么问题。接下来，我们再看看InnoDB怎么解决幻读的问题。

### 解决幻读

现在你知道了，产生幻读的原因是，行锁只能锁住行，但是新插入记录这个动作，要更新的是记录之间的间隙。因此，为了解决幻读问题，InnoDB只好引入新的锁，也就是间隙锁(Gap Lock)。顾名思义，间隙锁，锁的就是两个值之间的空隙。比如文章开头的表t，初始化插入了6个记录，这就产生了7个间隙。

![](E:\GongZuoQu\KTZhiShiKu\Image\GeekBang\MySQLShiZhan\HuanDu_img02.png)

这样，当你执行select * from t  where d=5 for update的时候，就不止是给数据库中已有的6个记录加上了行锁，还同时加了7个间隙锁。这样就确保了无法再插入新的记录。也就是说这时候，在一行行扫描的过程中，不仅将给行加上了行锁，还给行两边的空隙，也加上了间隙锁。

现在你知道了，数据行是可以加上锁的实体，数据行之间的间隙，也是可以加上锁的实体。但是间隙锁跟我们之前碰到过的锁都不太一样。比如行锁，分成读锁和写锁。下表就是这两种类型行锁的冲突关系。

|      | 读锁 | 写锁 |
| ---- | ---- | ---- |
| 读锁 | 兼容 | 冲突 |
| 写锁 | 冲突 | 冲突 |

也就是说，跟行锁有冲突关系的是另外一个行锁。但是间隙锁不一样，跟间隙锁存在冲突关系的，是往这个间隙中插入一个记录这个操作。间隙锁之间都不存在冲突关系。这句话不太好理解，我给你举个例子：

| sessionA                                      | sessionB                              |
| --------------------------------------------- | ------------------------------------- |
| begin;                                        |                                       |
| select * from t where c=7 lock in share mode; |                                       |
|                                               | begin;                                |
|                                               | select * from t where c=7 for update; |

这里sessionB并不会被堵住。因为表t里并没有c=7这个记录，因此sessionA加的是间隙锁(5,10)。而sessionB也是在这个间隙加的间隙锁。它们有共同的目标，即：保护这个间隙，不允许插入值。但，它们之间是不冲突的。

间隙锁和行锁合称next-key lock，每个next-key lock是前开后闭区间。也就是说，我们的表t初始化以后，如果用select * from t for update要把整个表所有记录锁起来，就形成了7个next-key lock，分别是(-∞,0]、(0,5]、(5,10]、(10,15]、(15,20]、(20,25]、(25,+supremum]。

> 备注：如果没有特别说明，我们把间隙锁记为开区间，把next-key lock记为前开后闭区间。

这个supremum从哪儿来的呢？这是因为+∞是开区间。实现上，InnoDB给每个索引加了一个不存在的最大值supremum，这样才符合我们前面说的都是前开后闭区间。

**间隙锁和next-key lock的引入，帮解决了幻读的问题，但同时也带来了一些困扰。**

在前面的文章中，就有同学提到了这个问题。我把他的问题转述一下，对应到我们这个例子的表来说，业务逻辑这样的：任意锁住一行，如果这一行不存在的话就插入，如果存在这一行就更新它的数据，代码如下：

```mysql
begin;
select * from t where id=N for update;
/*如果行不存在*/
insert into t values(N,N,N);
/*如果行存在*/
update t set d=N set id=N;
commit;
```

可能你会说，这个不是 insert … on duplicate key update 就能解决吗？但其实在有多个唯一键的时候，这个方法是不能满足这位提问同学的需求的。至于为什么，我会在后面的文章中再展开说明。现在，我们就只讨论这个逻辑。

这个同学碰到的现象是，这个逻辑一旦有并发，就会碰到死锁。你一定也觉得奇怪，这个逻辑每次操作前用for update锁起来，已经是最严格的模式了，怎么还会有死锁呢？这里，我用两个session来模拟并发，并假设N=9。

| sessionA                                                     | sessionB                               |
| ------------------------------------------------------------ | -------------------------------------- |
| begin;                                                       |                                        |
| select * from t where id=9 for update;                       |                                        |
|                                                              | begin;                                 |
|                                                              | select * from t where id=9 for update; |
|                                                              | insert into t values (9,9,9);(blocked) |
| insert into t values (9,9,9);<br>ERROR 1213(40001): Deadlock found |                                        |

其实都不需要用到后面的 update 语句，就已经形成死锁了。我们按语句执行顺序来分析一下：

- sessionA执行select …  for update语句，由于id=9这一行并不存在，因此会加上间隙锁(5,10);
- sessionB执行select … for  update语句，同样会加上间隙锁(5,10)，间隙锁之间不会冲突，因此这个语句可以执行成功；
- sessionB试图插入一行  (9,9,9)，被sessionA的间隙锁挡住了，只好进入等待；
- sessionA试图插入一行 (9,9,9)，被sessionB的间隙锁挡住了。

至此，两个session进入互相等待状态，形成死锁。当然，InnoDB的死锁检测马上就发现了这对死锁关系，让sessionA的insert语句报错返回了。你现在知道了，**间隙锁的引入，可能会导致同样的语句锁住更大的范围，这其实是影响了并发度的**。

你可能会说，为了解决幻读的问题，我们引入了这么一大串内容，有没有更简单一点的处理方法呢。我在文章一开始就说过，如果没有特别说明，今天和你分析的问题都是在可重复读隔离级别下的，间隙锁是在可重复读隔离级别下才会生效的。所以，你如果把隔离级别设置为读提交的话，就没有间隙锁了。但同时，你要解决可能出现的数据和日志不一致问题，需要把binlog格式设置为row。这，也是现在不少公司使用的配置组合。

表8：间隙锁导致的死锁；把innodb_locks_unsafe_for_binlog设置为1之后，sessionB并不会blocked，sessionA insert会阻塞住，但是不会提示死锁；然后sessionB提交执行成功，sessionA提示主键冲突。innodb_locks_unsafe_for_binlog这个参数就是这个意思不加gap lock，这个已经要被废弃了（8.0就没有了），所以不建议设置，容易造成误会。如果真的要去掉gap lock，可以考虑改用RC隔离级别加binlog_format=row。

前面文章的评论区有同学留言说，他们公司就使用的是读提交隔离级别加binlog_format=row的组合。他曾问他们公司的DBA说，你为什么要这么配置。DBA直接答复说，因为大家都这么用呀。所以，这个同学在评论区就问说，这个配置到底合不合理。关于这个问题本身的答案是，如果读提交隔离级别够用，也就是说，业务不需要可重复读的保证，这样考虑到读提交下操作数据的锁范围更小（没有间隙锁），这个选择是合理的。

但其实我想说的是，配置是否合理，跟业务场景有关，需要具体问题具体分析。但是，如果DBA认为之所以这么用的原因是大家都这么用，那就有问题了，或者说，迟早会出问题。比如说，大家都用读提交，可是逻辑备份的时候，mysqldump为什么要把备份线程设置成可重复读呢？

### 思考题

| sessionA                                                     | sessionB                         | sessionC                      |
| ------------------------------------------------------------ | -------------------------------- | ----------------------------- |
| begin;                                                       |                                  |                               |
| select * from t where c>=15 and c<=20 order by c desc for update; |                                  |                               |
|                                                              | insert into t values (11,11,11); |                               |
|                                                              |                                  | insert into t values (6,6,6); |

如果你之前没有了解过本篇文章的相关内容，一定觉得这三个语句简直是风马牛不相及。但实际上，这里sessionB和sessionC的insert语句都会进入锁等待状态。你可以试着分析一下，出现这种情况的原因是什么？这里需要说明的是，这其实是我在下一篇文章介绍加锁规则后才能回答的问题。
