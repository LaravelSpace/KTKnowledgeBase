```json
{
  "updated_by": "KelipuTe",
  "updated_at": "2020-07-05",
  "tags": "极客时间,GeekBang,MySQL实战45讲,MySQL"
}
```

---

## 全局锁和表锁

数据库锁设计的初衷是处理并发问题。作为多用户共享的资源，当出现并发访问的时候，数据库需要合理地控制资源的访问规则。而锁就是用来实现这些访问规则的重要数据结构。根据加锁的范围，MySQL里面的锁大致可以分成**全局锁、表级锁和行锁**三类。这里需要说明的是，锁的设计比较复杂，这里主要介绍的是碰到锁时的现象和其背后的原理。

### 全局锁

顾名思义，全局锁就是对整个数据库实例加锁。MySQL提供了一个加全局读锁的方法，命令是 `Flush tables with read lock`(FTWRL)。当你需要让整个库处于只读状态的时候，可以使用这个命令，之后其他线程的以下语句会被阻塞：数据更新语句（数据的增删改）、数据定义语句（包括建表、修改表结构等）和更新类事务的提交语句。

FTWRL前有读写的话，FTWRL都会等待读写执行完毕后才执行。FTWRL执行的时候要刷脏页的数据到磁盘，因为要保持数据的一致性，理解的执行FTWRL时候是所有事务都提交完毕的时候。这里有个脏页的概念，MySQL脏页是指，当内存数据页和磁盘数据页上的内容不一致时，我们称这个内存页为脏页。内存数据写入磁盘后，内存页上的数据和磁盘页上的数据就一致了，我们称这个内存页为干净页。

全局锁的典型使用场景是，做全库逻辑备份。也就是把整库每个表都select出来存成文本。以前有一种做法，是通过FTWRL确保不会有其他线程对数据库做更新，然后对整个库做备份。注意，在备份过程中整个库完全处于只读状态。但是让整库都只读，听上去就很危险：如果你在主库上备份，那么在备份期间都不能执行更新，业务基本上就得停摆；如果你在从库上备份，那么备份期间从库不能执行主库同步过来的binlog，会导致主从延迟。

看来加全局锁不太好。但是细想一下，备份为什么要加锁呢？我们来看一下不加锁会有什么问题。假设你现在要维护极客时间的购买系统，关注的是用户账户余额表和用户课程表。现在发起一个逻辑备份。假设备份期间，有一个用户，他购买了一门课程，业务逻辑里就要扣掉他的余额，然后往已购课程里面加上一门课。如果时间顺序上是先备份账户余额表(u_account)，然后用户购买，然后备份用户课程表(u_course)，会怎么样呢？你可以看一下这个图：

![](E:\Workspace\KTKnowledgeBase\Image\GeekBang\MySQLShiZhan\QuanJuSuoHeBiaoSuo_img01.png)

可以看到，这个备份结果里，用户A的数据状态是账户余额没扣，但是用户课程表里面已经多了一门课。如果后面用这个备份来恢复数据的话，用户A就发现，自己赚了。作为用户可别觉得这样可真好啊，你可以试想一下：如果备份表的顺序反过来，先备份用户课程表再备份账户余额表，又可能会出现什么结果？

也就是说，不加锁的话，备份系统备份的得到的库不是一个逻辑时间点，这个视图是逻辑不一致的。说到视图你肯定想起来了，我们在前面讲事务隔离的时候，其实是有一个方法能够拿到一致性视图的，对吧？是的，就是在可重复读隔离级别下开启一个事务。

官方自带的逻辑备份工具是mysqldump。当mysqldump使用参数–single-transaction的时候，导数据之前就会启动一个事务，来确保拿到一致性视图。而由于MVCC的支持，这个过程中数据是可以正常更新的。你一定在疑惑，有了这个功能，为什么还需要FTWRL呢？

**一致性读是好，但前提是引擎要支持这个隔离级别**。比如，对于MyISAM这种不支持事务的引擎，如果备份过程中有更新，总是只能取到最新的数据，那么就破坏了备份的一致性。这时，我们就需要使用FTWRL命令了。所以，**single-transaction方法只适用于所有的表使用事务引擎的库**。如果有的表使用了不支持事务的引擎，那么备份就只能通过FTWRL方法。这往往是DBA要求业务开发人员使用InnoDB替代MyISAM的原因之一。

你也许会问，既然要全库只读，为什么不使用 `set global readonly=true` 的方式呢？确实readonly方式也可以让全库进入只读状态，但我还是会建议你用FTWRL方式，主要有两个原因：

- 一是，在有些系统中，readonly的值会被用来做其他逻辑，比如用来判断一个库是主库还是备库。因此，修改global变量的方式影响面更大，我不建议你使用。
- 二是，在异常处理机制上有差异。如果执行FTWRL命令之后由于客户端发生异常断开，那么MySQL会自动释放这个全局锁，整个库回到可以正常更新的状态。而将整个库设置为readonly之后，如果客户端发生异常，则数据库就会一直保持readonly状态，这样会导致整个库长时间处于不可写状态，风险较高。全库只读readonly=true还有个情况，在slave上，如果用户有super权限的话readonly是失效的。

业务的更新不只是增删改数据（DML)，还有可能是加字段等修改表结构的操作（DDL）。不论是哪种方法，一个库被全局锁上以后，你要对里面任何一个表做加字段操作，都是会被锁住的。但是，即使没有被全局锁住，加字段也不是就能一帆风顺的，因为你还会碰到接下来我们要介绍的表级锁。

### 表级锁

MySQL里面表级别的锁有两种：一种是表锁，一种是元数据锁（meta data lock，MDL)。表锁的语法是**lock tables…read/write**。与FTWRL类似，可以用unlock tables主动释放锁，也可以在客户端断开的时候自动释放。需要注意，lock tables语法除了会限制别的线程的读写外，也限定了本线程接下来的操作对象。

举个例子，如果在某个线程A中执行 `lock tables t1 read,t2 write;` 这个语句，则其他线程写t1、读写t2的语句都会被阻塞。同时，线程A在执行unlock tables之前，也只能执行读t1、读写t2的操作。连写t1都不允许，自然也不能访问其他表。在还没有出现更细粒度的锁的时候，表锁是最常用的处理并发的方式。而对于InnoDB这种支持行锁的引擎，一般不使用lock tables命令来控制并发，毕竟锁住整个表的影响面还是太大。

另一类表级的锁是MDL（meta data lock)。MDL不需要显式使用，在访问一个表的时候会被自动加上。MDL的作用是，保证读写的正确性。你可以想象一下，如果一个查询正在遍历一个表中的数据，而执行期间另一个线程对这个表结构做变更，删了一列，那么查询线程拿到的结果跟表结构对不上，肯定是不行的。

因此，在MySQL5.5版本中引入了MDL，当对一个表做增删改查操作的时候，加MDL读锁；当要对表做结构变更操作的时候，加MDL写锁。读锁之间不互斥，因此你可以有多个线程同时对一张表增删改查。读写锁之间、写锁之间是互斥的，用来保证变更表结构操作的安全性。因此，如果有两个线程要同时给一个表加字段，其中一个要等另一个执行完才能开始执行。

虽然MDL锁是系统默认会加的，但却是你不能忽略的一个机制。比如下面这个例子，我经常看到有人掉到这个坑里：给一个小表加个字段，导致整个库挂了。你肯定知道，给一个表加字段，或者修改字段，或者加索引，需要扫描全表的数据。在对大表操作的时候，你肯定会特别小心，以免对线上服务造成影响。而实际上，即使是小表，操作不慎也会出问题。我们来看一下下面的操作序列，假设表t是一个小表。

备注：这里的实验环境是MySQL5.6。

| sessionA                 | sessionB                 | sessionC                          | sessionD                          |
| ------------------------ | ------------------------ | --------------------------------- | --------------------------------- |
| begin;                   |                          |                                   |                                   |
| select * from t limit 1; |                          |                                   |                                   |
|                          | select * from t limit 1; |                                   |                                   |
|                          |                          | alter table t add f int;(blocked) | select * from t limit 1;(blocked) |

我们可以看到sessionA先启动，这时候会对表t加一个MDL读锁。由于sessionB需要的也是MDL读锁，因此可以正常执行。之后sessionC会被blocked，是因为sessionA的MDL读锁还没有释放，而sessionC需要MDL写锁，因此只能被阻塞。如果只有sessionC自己被阻塞还没什么关系，但是之后所有要在表t上新申请MDL读锁的请求也会被sessionC阻塞。前面我们说了，所有对表的增删改查操作都需要先申请MDL读锁，就都被锁住，等于这个表现在完全不可读写了。

如果某个表上的查询语句频繁，而且客户端有重试机制，也就是说超时后会再起一个新session再请求的话，这个库的线程很快就会爆满。你现在应该知道了，事务中的MDL锁，在语句执行开始时申请，但是语句结束后并不会马上释放，而会等到整个事务提交后再释放。基于上面的分析，我们来讨论一个问题，如何安全地给小表加字段？

首先我们要解决长事务，事务不提交，就会一直占着MDL锁。在MySQL的information_schema库的innodb_trx表中，你可以查到当前执行中的事务。如果你要做DDL变更的表刚好有长事务在执行，要考虑先暂停DDL，或者kill掉这个长事务。但考虑一下这个场景。如果你要变更的表是一个热点表，虽然数据量不大，但是上面的请求很频繁，而你不得不加个字段，你该怎么做呢？

这时候kill可能未必管用，因为新的请求马上就来了。比较理想的机制是，在alter table语句里面设定等待时间，如果在这个指定的等待时间里面能够拿到MDL写锁最好，拿不到也不要阻塞后面的业务语句，先放弃。之后开发人员或者DBA再通过重试命令重复这个过程。MariaDB已经合并了AliSQL的这个功能，所以这两个开源分支目前都支持DDL NOWAIT/WAIT n这个语法。

```sql
ALTER TABLE tbl_name NOWAIT add column ...
ALTER TABLE tbl_name WAIT N add column ... 
```

### Online DDL

MySQL5.6支持Online DDL。也就是对表操作增加字段等功能。Online DDL的过程是这样的：1、拿MDL写锁。2、降级成MDL读锁。3、真正做DDL。4、升级成MDL写锁。5、释放MDL锁。1、2、4、5如果没有锁冲突，执行时间非常短。第3步占用了DDL绝大部分时间，这期间这个表可以正常读写数据，是因此称为online。

Online DDL的两个子选项包括ALGORITHM和LOCK：对于ALGORITHM参数使用default默认值即可，不需要强制指定该值，系统会自行判断，优先使用inplace，对于不支持的表或DDL操作使用copy。LOCK参数绝大多数情况下也不需要显式指定值，默认值default已经是尽可能允许DML的并行操作了。

inplace(rebuild)的整体执行过程如下：

准备阶段：

- 1、对表加元数据共享升级锁，并升级为排他锁。（此时DML不能并行）
- 2、在原表所在的路径下创建.frm和.ibd临时中转文件（no-rebuild除创建二级索引外只创建.frm文件，其中添加二级索引操作最为特殊，该操作属于no-rebuild不会生成.ibd，但实际上对.ibd文件却做了修改，该操作会在参数tmpdir指定路径下生成临时文件，用于存储索引排序结果，然后再合并到.ibd文件中）
- 3、申请row log空间，用于存放DDL执行阶段产生的DML操作。（no-rebuild不需要）

执行阶段：

- 1、释放排他锁，保留元数据共享升级锁（此时DML可以并行）。
- 2、扫描原表主键以及二级索引的所有数据页，生成B+树，存储到临时文件中；
- 3、将所有对原表的DML操作记录在日志文件row log中。注：如果只修改元数据部分（no-rebuild）该阶段只是修改.frm文件，不需要其他操作，也不需要申请row log。

提交阶段：

- 1、升级元数据共享升级锁，产生排他锁锁表（此时DML不能并行）。
- 2、重做row log中的内容。（no-rebuild不需要）
- 3、重命名原表文件，将临时文件改名为原表文件名，删除原表文件。
- 4、提交事务，变更完成。

copy的整体执行过程如下：

- 1、锁表，期间DML不可并行执行。
- 2、生成临时表以及临时表文件（.frm，.ibd）。
- 3、拷贝原表数据到临时表。
- 4、重命名临时表及文件。
- 5、删除原表及文件。
- 6、提交事务，释放锁。

### 思考题

备份一般都会在备库上执行，你在用–single-transaction方法做逻辑备份的过程中，如果主库上的一个小表做了一个DDL，比如给一个表上加了一列。这时候，从备库上会看到什么现象呢？

假设这个DDL是针对表t1的， 这里我把备份过程中几个关键的语句列出来：

```mysql
Q1:SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
Q2:START TRANSACTION  WITH CONSISTENT SNAPSHOT；
/* other tables */
Q3:SAVEPOINT sp;
/* 时刻 1 */
Q4:show create table `t1`;
/* 时刻 2 */
Q5:SELECT * FROM `t1`;
/* 时刻 3 */
Q6:ROLLBACK TO SAVEPOINT sp;
/* 时刻 4 */
/* other tables */
```

在备份开始的时候，为了确保 RR（可重复读）隔离级别，再设置一次 RR 隔离级别 (Q1)；

启动事务，这里用 WITH CONSISTENT SNAPSHOT 确保这个语句执行完就可以得到一个一致性视图（Q2)；

设置一个保存点，这个很重要（Q3）；

show create 是为了拿到表结构  (Q4)，然后正式导数据 （Q5），回滚到 SAVEPOINT sp，在这里的作用是释放 t1 的 MDL 锁  （Q6）。当然这部分属于“超纲”，上文正文里面都没提到。

DDL  从主库传过来的时间按照效果不同，我打了四个时刻。题目设定为小表，我们假定到达后，如果开始执行，则很快能够执行完成。

参考答案如下：

1、如果在 Q4  语句执行之前到达，现象：没有影响，备份拿到的是 DDL 后的表结构。

2、如果在“时刻 2”到达，则表结构被改过，Q5 执行的时候，报 Table  definition has changed, please retry transaction，现象：mysqldump 终止；

3、如果在“时刻  2”和“时刻 3”之间到达，mysqldump 占着 t1 的 MDL 读锁，binlog 被阻塞，现象：主从延迟，直到 Q6  执行完成。

4、从“时刻 4”开始，mysqldump 释放了 MDL 读锁，现象：没有影响，备份拿到的是 DDL 前的表结构。
