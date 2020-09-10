```json
{
  "updated_by": "KelipuTe",
  "updated_at": "2020-08-14",
  "tags": "极客时间,GeekBang,MySQL实战45讲,MySQL"
}
```

---

## 数据可靠性

依靠WAL机制，只要redo log和binlog保证持久化到磁盘，就能确保MySQL异常重启后，数据可以恢复。

### binlog的写入机制

binlog的写入逻辑比较简单：事务执行过程中，先把日志写到binlog cache，事务提交的时候，再把binlog cache写到binlog文件中。一个事务的binlog是不能被拆开的，因此不论这个事务多大，也要确保一次性写入。这就涉及到了binlog cache的保存问题。

系统给binlog cache分配了一片内存，每个线程一个，参数binlog_cache_size用于控制单个线程内binlog cache所占内存的大小。如果超过了这个参数规定的大小，就要暂存到磁盘。事务提交的时候，执行器把binlog cache里的完整事务写入到binlog中，并清空binlog cache。状态如图所示。

![](E:\GongZuoQu\KTZhiShiKu\Image\GeekBang\MySQLShiZhan\ShuJuKeKaoXing_img02.png)

可以看到，每个线程有自己binlog cache，但是共用同一份binlog文件。

- 图中的write，指的就是指把日志写入到文件系统的page cache，并没有把数据持久化到磁盘，所以速度比较快。
- 图中的fsync，才是将数据持久化到磁盘的操作。一般情况下，我们认为fsync才占磁盘的IOPS。

write和fsync的时机，是由参数sync_binlog控制的：

- sync_binlog=0的时候，表示每次提交事务都只write，不fsync；
- sync_binlog=1的时候，表示每次提交事务都会执行fsync；
- sync_binlog=N(N>1)的时候，表示每次提交事务都write，但累积N个事务后才fsync。

因此，在出现IO瓶颈的场景里，将sync_binlog设置成一个比较大的值，可以提升性能。在实际的业务场景中，考虑到丢失日志量的可控性，一般不建议将这个参数设成0，比较常见的是将其设置为100~1000中的某个数值。但是，将sync_binlog设置为N，对应的风险是：如果主机发生异常重启，会丢失最近N个事务的binlog日志。

但是设置sync_binlog=0的时候，还是可以看到binlog文件马上做了修改。这个是对的，我们说把日志写到了page cache，这就是文件系统的page cache。而用ls命令看到的就是文件系统返回的结果。

### redo log的写入机制

redo log的写入机制相对复杂。事务在执行过程中，生成的redo log是要先写到redo log buffer的。redo log buffer里面的内容，不需要每次生成后都直接持久化到磁盘。如果事务执行期间MySQL发生异常重启，那这部分日志就丢了。由于事务并没有提交，所以这时日志丢了也不会有损失。

事务还没提交的时候，redo log buffer中的部分日志会有可能被持久化到磁盘。这个问题，要从redo log可能存在的三种状态说起。这三种状态，对应的就是图中的三个颜色块。

![](E:\GongZuoQu\KTZhiShiKu\Image\GeekBang\MySQLShiZhan\ShuJuKeKaoXing_img04.png)

这三种状态分别是：

- 存在redo log buffer中，物理上是在MySQL进程内存中，就是图中的红色部分。
- 写到磁盘(write)，但是没有持久化（fsync)，物理上是在文件系统的page cache里面，也就是图中的黄色部分。
- 持久化到磁盘，对应的是hard disk，也就是图中的绿色部分。

日志写到redo log buffer是很快的，wirte到page cache也差不多，但是持久化到磁盘的速度就慢多了。为了控制redo log的写入策略，InnoDB提供了innodb_flush_log_at_trx_commit参数，它有三种可能取值：

- 设置为0的时候，表示每次事务提交时都只是把redo log留在redo log buffer中。
- 设置为1的时候，表示每次事务提交时都将redo log直接持久化到磁盘。
- 设置为2的时候，表示每次事务提交时都只是把redo log写到page cache。

InnoDB有一个后台线程，每隔1秒，就会把redo log buffer中的日志，调用write写到文件系统的page cache，然后调用fsync持久化到磁盘。注意，事务执行中间过程的redo log也是直接写在redo log buffer中的，这些redo log也会被后台线程一起持久化到磁盘。也就是说，一个没有提交的事务的redo log，也是可能已经持久化到磁盘的。

实际上，除了后台线程每秒一次的轮询操作外，还有两种场景会让一个没有提交的事务的redo log写入到磁盘中。

- 一种是，redo log buffer占用的空间即将达到innodb_log_buffer_size一半的时候，后台线程会主动写盘。注意，由于这个事务并没有提交，所以这个写盘动作只是write，而没有调用fsync，也就是只留在了文件系统的page cache。
- 另一种是，并行的事务提交的时候，顺带将这个事务的redo log buffer持久化到磁盘。假设一个事务A执行到一半，已经写了一些redo log到buffer中，这时候有另外一个线程的事务B提交，如果innodb_flush_log_at_trx_commit设置的是1，那么按照这个参数的逻辑，事务B要把redo log buffer里的日志全部持久化到磁盘。这时候，就会带上事务A在redo log buffer里的日志一起持久化到磁盘。

这里需要说明的是，介绍两阶段提交的时候说过，时序上redo log先prepare，再写binlog，最后再把redo log commit。如果把innodb_flush_log_at_trx_commit设置成1，那么redo log在prepare阶段就要持久化一次，因为有一个崩溃恢复逻辑是要依赖于prepare的redo log，再加上binlog来恢复的。每秒一次后台轮询刷盘，再加上崩溃恢复这个逻辑，InnoDB就认为redo log在commit的时候就不需要fsync了，只会write到文件系统的page cache中就够了。

通常我们说MySQL的双1配置，指的就是sync_binlog和innodb_flush_log_at_trx_commit都设置成1。也就是说，一个事务完整提交前，需要等待两次刷盘，一次是redo log（prepare阶段），一次是binlog。这意味着我从MySQL看到的TPS是每秒两万的话，每秒就会写四万次磁盘。但是，用工具测试出来，磁盘能力也就两万左右，解释这个问题，就要用到组提交（group commit）机制了。

这里，需要先介绍日志逻辑序列号（log sequence number，LSN）的概念。LSN是单调递增的，用来对应redo log的一个个写入点。每次写入长度为length的redo log，LSN的值就会加上length。LSN也会写到InnoDB的数据页中，来确保数据页不会被多次执行重复的redo log。如图所示，是三个并发事务(trx1,trx2,trx3)在prepare阶段，都写完redo log buffer，持久化到磁盘的过程，对应的LSN分别是50、120和160。

![](E:\GongZuoQu\KTZhiShiKu\Image\GeekBang\MySQLShiZhan\ShuJuKeKaoXing_img06.png)

从图中可以看到，

- trx1是第一个到达的，会被选为这组的leader。
- 等trx1要开始写盘的时候，这个组里面已经有了三个事务，这时候LSN也变成了160。
- trx1去写盘的时候，带的就是LSN=160，因此等trx1返回时，所有LSN小于等于160的redo log，都已经被持久化到磁盘。
- 这时候trx2和trx3就可以直接返回了。

所以，一次组提交里面，组员越多，节约磁盘IOPS的效果越好。但如果只有单线程压测，那就只能老老实实地一个事务对应一次持久化操作了。在并发更新场景下，第一个事务写完redo log buffer以后，接下来这个fsync越晚调用，组员可能越多，节约IOPS的效果就越好。为了让一次fsync带的组员更多，MySQL有一个很有趣的优化：拖时间。

![](E:\GongZuoQu\KTZhiShiKu\Image\GeekBang\MySQLShiZhan\ShuJuKeKaoXing_img08.png)

图中，把写binlog被当成一个动作。但实际上，写binlog是分成两步的：1、先把binlog从binlog cache中写到磁盘上的binlog文件。2、调用fsync持久化。MySQL为了让组提交的效果更好，把redo log做fsync的时间拖到了步骤1之后。也就是说，上面的图变成了这样：

![](E:\GongZuoQu\KTZhiShiKu\Image\GeekBang\MySQLShiZhan\ShuJuKeKaoXing_img10.png)

这么一来，binlog也可以组提交了。在执行图中第4步把binlog fsync到磁盘时，如果有多个事务的binlog已经写完了，也是一起持久化的，这样也可以减少IOPS的消耗。不过通常情况下第3步执行得会很快，所以binlog的write和fsync间的间隔时间短，导致能集合到一起持久化的binlog比较少，因此binlog的组提交的效果通常不如redo log的效果那么好。

如果想提升binlog组提交的效果，可以通过设置binlog_group_commit_sync_delay和binlog_group_commit_sync_no_delay_count来实现。binlog_group_commit_sync_delay参数，表示延迟多少微秒后才调用fsync。binlog_group_commit_sync_no_delay_count参数，表示累积多少次以后才调用fsync。

这两个条件是或的关系，也就是说只要有一个满足条件就会调用fsync。所以，当binlog_group_commit_sync_delay设置为0的时候，binlog_group_commit_sync_no_delay_count也无效了。

如果是在sync_binlog=0的情况下，设置binlog_group_commit_sync_delay和binlog_group_commit_sync_no_delay_count。binlog_group_commit_sync_delay和binlog_group_commit_sync_no_delay_count的逻辑先走，因此该等还是会等。等到满足了这两个条件之一，就进入sync_binlog阶段。这时候如果判断sync_binlog=0，就直接跳过，还是不调fsync。

WAL机制是减少磁盘写，可是每次提交事务都要写redo log和binlog，磁盘读写次数并没有变少。WAL机制主要得益于两个方面：1、redo log和binlog都是顺序写，磁盘的顺序写比随机写速度要快。2、组提交机制，可以大幅度降低磁盘的IOPS消耗。

如果MySQL出现了性能瓶颈，而且瓶颈在IO上，针对这个问题，可以考虑以下三种方法来。

- 设置binlog_group_commit_sync_delay和binlog_group_commit_sync_no_delay_count参数，减少binlog的写盘次数。这个方法是基于额外的故意等待来实现的，因此可能会增加语句的响应时间，但没有丢失数据的风险。
- 将sync_binlog设置为大于1的值（比较常见是100~1000）。这样做的风险是，主机掉电时会丢binlog日志。
- 将innodb_flush_log_at_trx_commit设置为2。这样做的风险是，主机掉电的时候会丢数据。

不建议你把innodb_flush_log_at_trx_commit设置成0。因为把这个参数设置成0，表示redo log只保存在内存中，这样的话MySQL本身异常重启也会丢数据，风险太大。而redo log写到文件系统的page cache的速度也是很快的，所以将这个参数设置成2跟设置成0其实性能差不多，但这样做MySQL异常重启时就不会丢数据了，相比之下风险会更小。

非双1配置在实际使用中也有可能出现。一般情况下，把生产库改成非双1配置，是设置innodb_flush_logs_at_trx_commit=2，sync_binlog=1000。可能有以下几个场景：

- 1、业务高峰期。一般如果有预知的高峰期，DBA会有预案，把主库设置成非双1。
- 2、备库延迟，为了让备库尽快赶上主库。
- 3、用备份恢复主库的副本，应用binlog的过程，这个跟上一种场景类似。
- 4、批量导入数据的时候。

场景2这里有一个有趣的现象，由于从库设置了binlog_group_commit_sync_delay和binlog_group_commit_sync_no_delay_count导致一直延迟的情况。在主库设置这两个参数，是为了减少binlog的写盘压力。备库这么设置，尤其在快要追上的时候，就反而会受这两个参数的拖累。一般追主备就用非双1（追上记得改回来）。

