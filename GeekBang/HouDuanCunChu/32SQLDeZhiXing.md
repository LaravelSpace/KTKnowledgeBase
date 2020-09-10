```json
{
  "updated_by": "KelipuTe",
  "updated_at": "2020-08-04",
  "tags": "极客时间,GeekBang,后端存储实践课"
}
```

---

## SQL是如何在数据库中执行的

数据库是一个非常非常复杂的软件系统，数据库的服务端，可以划分为执行器(Execution Engine)和存储引擎(Storage Engine)两部分。执行器负责解析SQL执行查询，存储引擎负责保存数据。

### 执行器执行 如何SQL

```sql
SELECT u.id AS user_id, u.name AS user_name, o.id AS order_id
FROM users u INNER JOIN orders o ON u.id = o.user_id WHERE u.id > 50
```

这个SQL语义是，查询用户ID大于50的用户的所有订单，这是很简单的一个联查，需要查询users和orders两张表，WHERE条件就是，用户ID大于50。

数据库收到查询请求后，需要先解析SQL语句，把这一串文本解析成便于程序处理的结构化数据，这就是一个通用的语法解析过程。跟编程语言的编译器编译时，解析源代码的过程是完全一样的。转换后的结构化数据，就是一棵树，这个树的名字叫抽象语法树（AST，Abstract Syntax Tree）。上面这个SQL，它的AST大概是这样的，只画了主要的部分：

![](E:\GongZuoQu\KTZhiShiKu\Image\GeekBang\HouDuanCunChu\SQLDeZhiXing_img01.png)

执行器解析这个AST之后，会生成一个逻辑执行计划。所谓的执行计划，可以简单理解为如何一步一步地执行查询和计算，最终得到执行结果的一个分步骤的计划。这个逻辑执行计划是这样的：

```
LogicalProject(user_id=[$0], user_name=[$1], order_id=[$5])
    LogicalFilter(condition=[$0 > 50])
        LogicalJoin(condition=[$0 == $6], joinType=[inner])
            LogicalTableScan(table=[users])
            LogicalTableScan(table=[orders])
```

和SQL、AST不同的是，这个逻辑执行计划已经很像可以执行的程序代码了。你看上面这个执行计划，很像我们编程语言的函数调用栈，外层的方法调用内层的方法。所以，要理解这个执行计划，得从内往外看。

- 1、最内层的2个LogicalTableScan的含义是，把USERS和ORDERS这两个表的数据都读出来。
- 2、然后拿这两个表所有数据做一个LogicalJoin，JOIN的条件就是第0列(u.id)等于第6列(o.user_id)。
- 3、然后再执行一个LogicalFilter过滤器，过滤条件是第0列(u.id)大于50。
- 4、最后，做一个LogicalProject投影，只保留第0(user_id)、1(user_name)、5(order_id)三列。这里投影(Project)的意思是，把不需要的列过滤掉。

把这个逻辑执行计划翻译成代码，然后按照顺序执行，就可以正确地查询出数据了。但是，按照上面那个执行计划，需要执行2个全表扫描，然后再把2个表的所有数据做一个JOIN操作，这个性能是非常差的。这种从SQL的AST直译过来的逻辑执行计划，一般性能都非常差，所以，需要对执行计划进行优化。

如何对执行计划进行优化，不同的数据库有不同的优化方法，这一块儿也是不同数据库性能有差距的主要原因之一。优化的总体思路是，在执行计划中，尽早地减少必须处理的数据量。也就是说，尽量在执行计划的最内层减少需要处理的数据量。看一下简单优化后的逻辑执行计划：

```
LogicalProject(user_id=[$0], user_name=[$1], order_id=[$5])
    LogicalJoin(condition=[$0 == $6], joinType=[inner])
        LogicalProject(id=[$0], name=[$1])              // 尽早执行投影
            LogicalFilter(condition=[$0 > 50])          // 尽早执行过滤
                LogicalTableScan(table=[users])
        LogicalProject(id=[$0], user_id=[$1])           // 尽早执行投影
            LogicalTableScan(table=[orders])
```

对比原始的逻辑执行计划，这里我们做了两点简单的优化：1、尽早地执行投影，去除不需要的列；2、尽早地执行数据过滤，去除不需要的行。这样，就可以在做JOIN之前，把需要JOIN的数据尽量减少。这个优化后的执行计划，显然会比原始的执行计划快很多。

到这里，执行器只是在逻辑层面分析SQL，优化查询的执行逻辑，我们执行计划中操作的数据，仍然是表、行和列。在数据库中，表、行、列都是逻辑概念，所以，这个执行计划叫逻辑执行计划。执行查询接下来的部分，就需要涉及到数据库的物理存储结构了。

###  存储引擎如何执行SQL

数据真正存储的时候，无论在磁盘里，还是在内存中，都没法直接存储这种带有行列的二维表。数据库中的二维表，实际上是怎么存储的呢？这就是存储引擎负责解决的问题，存储引擎主要功能就是把逻辑的表行列，用合适的物理存储结构保存到文件中。不同的数据库，它们的物理存储结构是完全不一样的，这也是各种数据库之间巨大性能差距的根本原因。

我们还是以MySQL为例来说一下它的物理存储结构。MySQL非常牛的一点是，它在设计层面对存储引擎做了抽象，它的存储引擎是可以替换的。它默认的存储引擎是InnoDB，在InnoDB中，数据表的物理存储结构是以主键为关键字的B+树，每一行数据直接就保存在B+树的叶子节点上。比如，上面的订单表组织成B+树，是这个样的：

![](E:\GongZuoQu\KTZhiShiKu\Image\GeekBang\HouDuanCunChu\SQLDeZhiXing_img02.png)

这个树以订单表的主键orders.id为关键字组织，其中62:[row data]，表示的是订单号为62的一行订单数据。在InnoDB中，表的索引也是以B+树的方式来存储的，和存储数据的B+树的区别是，在索引树中，叶子节点保存的不是行数据，而是行的主键值。如果通过索引来检索一条记录，需要先后查询索引树和数据树这两棵树：先在索引树中检索到行记录的主键值，然后再用主键值去数据树中去查找这一行数据。

优化后的逻辑执行计划将会被转换成物理执行计划，物理执行计划是和数据的物理存储结构相关的。还是用InnoDB来举例，直接将逻辑执行计划转换为物理执行计划：

```
InnodbProject(user_id=[$0], user_name=[$1], order_id=[$5])
    InnodbJoin(condition=[$0 == $6], joinType=[inner])
        InnodbTreeNodesProject(id=[key], name=[data[1]])
            InnodbFilter(condition=[key > 50])
                InnodbTreeScanAll(tree=[users])
        InnodbTreeNodesProject(id=[key], user_id=[data[1]])
            InnodbTreeScanAll(tree=[orders])
```

物理执行计划同样可以根据数据的物理存储结构、是否存在索引以及数据多少等各种因素进行优化。这一块儿的优化规则同样是非常复杂的，比如，我们可以把对用户树的全树扫描再按照主键过滤这两个步骤，优化为对树的范围查找。

```

PhysicalProject(user_id=[$0], user_name=[$1], order_id=[$5])
    PhysicalJoin(condition=[$0 == $6], joinType=[inner])
        InnodbTreeNodesProject(id=[key], name=[data[1]])
            InnodbTreeRangeScan(tree=[users], range=[key > 50])  // 全树扫描再按照主键过滤，直接可以优化为对树的范围查找
        InnodbTreeNodesProject(id=[key], user_id=[data[1]])
            InnodbTreeScanAll(tree=[orders])
```

最终，按照优化后的物理执行计划，一步一步地去执行查找和计算，就可以得到SQL的查询结果了。理解数据库执行SQL的过程，以及不同存储引擎中的数据和索引的物理存储结构，对于正确使用和优化SQL非常有帮助。

比如，我们知道了InnoDB的索引实现后，就很容易明白为什么主键不能太长，因为表的每个索引保存的都是主键的值，过长的主键会导致每一个索引都很大。而且，主键除了不能太长之外最好能有序，有序的话能减少插入时B+树的排序操作，所以uuid这种其实不适合做主键。

再比如，我们了解了执行计划的优化过程后，就很容易理解，有的时候明明有索引却不能命中的原因是，数据库在对物理执行计划优化的时候，评估发现不走索引，直接全表扫描是更优的选择。

