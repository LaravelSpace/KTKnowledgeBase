```json
{
  "updated_by": "KelipuTe",
  "updated_at": "2020-07-05",
  "tags": "极客时间,GeekBang,MySQL实战45讲,MySQL"
}
```

---

## 事务隔离

简单来说，事务就是要保证一组数据库操作，要么全部成功，要么全部失败。在MySQL中，事务支持是在引擎层实现的。你现在知道，MySQL是一个支持多引擎的系统，但并不是所有的引擎都支持事务。比如MySQL原生的MyISAM引擎就不支持事务，这也是MyISAM被InnoDB取代的重要原因之一。

### 隔离性与隔离级别

提到事务，你肯定会想到ACID（Atomicity、Consistency、Isolation、Durability，即原子性、一致性、隔离性、持久性），今天我们就来说说其中I，也就是隔离性。当数据库上有多个事务同时执行的时候，就可能出现脏读（dirty read）、不可重复读（non-repeatable read）、幻读（phantom read）的问题，为了解决这些问题，就有了隔离级别的概念。

- 脏读：当数据库中一个事务A正在修改一个数据但是还未提交或者回滚，另一个事务B来读取了修改后的内容并且使用了，之后事务A提交了，此时就引起了脏读。此情况仅会发生在：读未提交的的隔离级别。

- 不可重复读：在一个事务A中多次操作数据，在事务操作过程中(未最终提交)，事务B也才做了处理，并且该值发生了改变，这时候就会导致A在事务操作的时候，发现数据与第一次不一样了。就是不可重复读。此情况仅会发生在：读未提交、读提交的隔离级别。

- 幻读：一个事务按相同的查询条件重新读取以前检索过的数据，却发现其他事务插入了满足其查询条件的新数据，这种现象就称为幻读。

  幻读是指当事务不是独立执行时发生的一种现象，例如第一个事务对一个表中的数据进行了修改，比如这种修改涉及到表中的全部数据行。同时，第二个事务也修改这个表中的数据，这种修改是向表中插入一行新数据。那么，以后就会发生操作第一个事务的用户发现表中还存在没有修改的数据行，就好象发生了幻觉一样。此情况会回发生在：读未提交、读提交、可重复读的隔离级别。一般解决幻读的方法是增加范围锁，锁定检索范围为只读，这样就避免了幻读。

在谈隔离级别之前，你首先要知道，你隔离得越严实，效率就会越低。因此很多时候，我们都要在二者之间寻找一个平衡点。SQL标准的事务隔离级别包括：读未提交（read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（serializable）。这4种隔离级别，并行性能依次降低，安全性依次提高。

- 读未提交是指，一个事务还没提交时，它做的变更就能被别的事务看到。
- 读提交是指，一个事务提交之后，它做的变更才会被其他事务看到。
- 可重复读是指，一个事务执行过程中看到的数据，总是跟这个事务在启动时看到的数据是一致的。当然在可重复读隔离级别下，未提交变更对其他事务也是不可见的。
- 串行化，顾名思义是对于同一行记录，写会加写锁，读会加读锁。当出现读写锁冲突的时候，后访问的事务必须等前一个事务执行完成，才能继续执行。

其中读提交和可重复读比较难理解，所以我用一个例子说明这几种隔离级别。假设数据表T中只有一列，其中一行的值为1，下面是按照时间顺序执行两个事务的行为。

```mysql
mysql> create table T(c int) engine=InnoDB;
mysql> insert into T(c) values(1);
```

| 事务A        | 事务B       |
| ------------ | ----------- |
| 启动事务     | 启动事务    |
| 查询得到值1  | 查询得到值1 |
|              | 将1改成2    |
| 查询得到值V1 |             |
|              | 提交事务B   |
| 查询得到值V2 |             |
| 提交事务A    |             |
| 查询得到值V3 |             |

我们来看看在不同的隔离级别下，事务 A  会有哪些不同的返回结果，也就是图里面 V1、V2、V3 的返回值分别是什么。

- 若隔离级别是读未提交，则V1的值就是2。这时候事务B虽然还没有提交，但是结果已经被A看到了。因此，V2、V3也都是2。别人改数据的事务尚未提交，我在我的事务中也能读到。
- 若隔离级别是读提交，则V1是1，V2的值是2。事务B的更新在提交后才能被A看到。所以，V3的值也是2。别人改数据的事务已经提交，我在我的事务中才能读到。
- 若隔离级别是可重复读，则V1、V2是1，V3是2。之所以V2还是1，遵循的就是这个要求：事务在执行期间看到的数据前后必须是一致的。别人改数据的事务已经提交，我在我的事务中也不去读。
- 若隔离级别是串行化，则在事务B执行将1改成2的时候，会被锁住。直到事务A提交后，事务B才可以继续执行。所以从A的角度看，V1、V2值是1，V3的值是2。我的事务尚未提交，别人就别想改数据。

在实现上，数据库里面会创建一个视图，访问的时候以视图的逻辑结果为准。在可重复读隔离级别下，这个视图是在事务启动时创建的，整个事务存在期间都用这个视图。在读提交隔离级别下，这个视图是在每个SQL语句开始执行的时候创建的。这里需要注意的是，读未提交隔离级别下直接返回记录上的最新值，没有视图概念；而串行化隔离级别下直接用加锁的方式来避免并行访问。

我们可以看到在不同的隔离级别下，数据库行为是有所不同的。Oracle数据库的默认隔离级别其实就是读提交，因此对于一些从Oracle迁移到MySQL的应用，为保证数据库隔离级别的一致，你一定要记得将MySQL的隔离级别设置为读提交。配置的方式是，将启动参数transaction-isolation的值设置成READ-COMMITTED。你可以用show variables来查看当前的值。

```mysql
mysql> show variables like 'transaction_isolation';
+-----------------------+-----------------+
| Variable_name         | Value           |
+-----------------------+-----------------+
| transaction_isolation | REPEATABLE-READ |
+-----------------------+-----------------+
1 row in set, 1 warning (0.01 sec)
```

那什么时候需要可重复读的场景呢？假设你在管理一个个人银行账户表。一个表存了账户余额，一个表存了账单明细。到了月底你要做数据校对，也就是判断上个月的余额和当前余额的差额，是否与本月的账单明细一致。你一定希望在校对过程中，即使有用户发生了一笔新的交易，也不影响你的校对结果。这时候使用可重复读隔离级别就很方便。事务启动时的视图可以认为是静态的，不受其他事务更新的影响。

### 事务隔离的实现

理解了事务的隔离级别，我们再来看看事务隔离具体是怎么实现的。这里我们展开说明可重复读。在MySQL中，实际上每条记录在更新的时候都会同时记录一条回滚操作。记录上的最新值，通过回滚操作，都可以得到前一个状态的值。假设一个值从1被按顺序改成了2、3、4，在回滚日志里面就会有类似下面的记录。

![](E:\GongZuoQu\KTZhiShiKu\Image\GeekBang\MySQLShiZhan\ShiWuGeLi1_img01.png)

当前值是4，但是在查询这条记录的时候，不同时刻启动的事务会有不同的read-view。如图中看到的，在视图A、B、C里面，这一个记录的值分别是1、2、4，同一条记录在系统中可以存在多个版本，就是数据库的多版本并发控制（MVCC）。

对于read-view A，要得到1，就必须将当前值依次执行图中所有的回滚操作得到。即使现在有另外一个事务正在将4改成5，这个事务跟read-view A、B、C对应的事务是不会冲突的。回滚日志总不会一直保留，在不需要的时候会被删除。系统会判断，当系统里没有比这个回滚日志更早的read-view的时候，也就是没有事务再需要用到这些回滚日志时，回滚日志会被删除。

基于上面的说明，我们来讨论一下为什么建议你尽量不要使用长事务。长事务意味着系统里面会存在很老的事务视图。由于这些事务随时可能访问数据库里面的任何数据，所以这个事务提交之前，数据库里面它可能用到的回滚记录都必须保留，这就会导致大量占用存储空间。

在MySQL5.5及以前的版本，回滚日志是跟数据字典一起放在ibdata文件里的，即使长事务最终提交，回滚段被清理，文件也不会变小。我见过数据只有20GB，而回滚段有200GB的库。最终只好为了清理回滚段，重建整个库。除了对回滚段的影响，长事务还占用锁资源，也可能拖垮整个库。

### 事务的启动方式

如前面所述，长事务有这些潜在风险，我当然是建议你尽量避免。其实很多时候业务开发同学并不是有意使用长事务，通常是由于误用所致。MySQL的事务启动方式有以下几种：

- 1、显式启动事务语句，begin或start transaction。配套的提交语句是commit，回滚语句是rollback。
- 2、set autocommit=0，这个命令会将这个线程的自动提交关掉。意味着如果你只执行一个select语句，这个事务就启动了，而且并不会自动提交。这个事务持续存在直到你主动执行commit或rollback语句，或者断开连接。

有些客户端连接框架会默认连接成功后先执行一个set autocommit=0的命令。这就导致接下来的查询都在事务中，如果是长连接，就导致了意外的长事务。因此，我会建议你总是使用set autocommit=1，通过显式语句的方式来启动事务。

但是有的开发同学会纠结多一次交互的问题。对于一个需要频繁使用事务的业务，第二种方式每个事务在开始时都不需要主动执行一次begin，减少了语句的交互次数。如果你也有这个顾虑，我建议你使用commit work and chain语法。

在autocommit为1的情况下，用begin显式启动的事务，如果执行commit则提交事务。如果执行commit work and chain，则是提交事务并自动启动下一个事务，这样也省去了再次执行begin语句的开销。同时带来的好处是从程序开发的角度明确地知道每个语句是否处于事务中。

你可以在information_schema库的innodb_trx这个表中查询长事务，比如下面这个语句，用于查找持续时间超过60s的事务。

````mysql
select * from information_schema.innodb_trx where TIME_TO_SEC(timediff(now(),trx_started))>60
````

### 如何避免长事务

首先，从应用开发端来看：

- 1、确认是否使用了 set autocommit=0。把 MySQL 的 general_log 开起来，通过 general_log 的日志来确认。一般框架如果会设置这个值，也就会提供参数来控制行为，把它改成 1。
- 2、确认是否有不必要的只读事务。有些框架会习惯不管什么语句先用 begin/commit 框起来。但是也把好几个 select 语句放到了事务中。这种只读事务可以去掉。
- 3、业务连接数据库的时候，根据业务本身的预估，通过 SET MAX_EXECUTION_TIME 命令，来控制每个语句执行的最长时间，避免单个语句意外执行太长时间。

其次，从数据库端来看：

- 1、监控 information_schema.Innodb_trx 表，设置长事务阈值，超过就报警 / 或者 kill；
- 2、Percona 的 pt-kill 这个工具不错，推荐使用；
- 3、在业务功能测试阶段要求输出所有的 general_log，分析日志行为提前发现问题；
- 4、如果使用的是 MySQL  5.6 或者更新版本，把 innodb_undo_tablespaces 设置成 2（或更大的值）。如果真的出现大事务导致回滚段过大，这样设置后清理起来更方便。