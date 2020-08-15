```json
{
  "updated_by": "KelipuTe",
  "updated_at": "2020-08-15",
  "tags": "极客时间,GeekBang,MySQL实战45讲,MySQL"
}
```

---

## 主备切换

前面介绍过MySQL的主备复制，但都是一主一备的结构。大多数的互联网应用场景都是读多写少，因此负责的业务，在发展过程中很可能先会遇到读性能的问题。而在数据库层解决读性能问题，就要涉及到一主多从的架构。如图1所示，就是一个基本的一主多从结构。

![](E:\Workspace\KTKnowledgeBase\Image\GeekBang\MySQLShiZhan\ZhuBeiQieHuan_img02.png)

图中，虚线箭头表示的是主备关系，也就是A和A'互为主备，从库B、C、D指向的是主库A。一主多从的设置，一般用于读写分离，主库负责所有的写入和一部分读，其他的读请求则由从库分担。在一主多从架构下，主库发生故障，主备切换后的结果，如图2所示。

![](E:\Workspace\KTKnowledgeBase\Image\GeekBang\MySQLShiZhan\ZhuBeiQieHuan_img04.png)

相比于一主一备的切换流程，一主多从结构在切换完成后，A'会成为新的主库，从库B、C、D也要改接到A'。正是由于多了从库B、C、D重新指向的这个过程，所以主备切换的复杂性也相应增加了。