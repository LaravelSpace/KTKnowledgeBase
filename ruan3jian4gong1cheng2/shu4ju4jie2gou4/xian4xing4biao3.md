## 线性表(list)

零个或多个数据元素，一个数据元素可以由若干数据项组成

有限序列(有顺序)

元素有且只有一个前驱一个后继(第一个元素只有后继，最后一个元素只有前驱)

### 顺序存储结构

连续存储空间

依次存储数据元素

高效随机访问，计算地址

低效的插入和删除，移动数据

可以通过标记待删除的数据，改善删除操作的低效的元素移动问题

长度动态变化时，难以确定容量

内存空间碎片(大块线性表内存空间之间的小块内存空间)

#### 数组(array)

线性表结构

连续内存空间

相同类型

警惕访问越界

数组下标从0开始，是因为数组下标表示的是内存的偏移量

#### 容器

如Java的ArrayList等

好处是封装了数组操作，还可以动态扩容

数组空间不够时，动态扩容到1.5倍，然后把原来的数据移到新的位置

依然存在移动数据，所以要预估规模指定合理的大小

### CPU的读取策略

CPU从内存读取数据并不是只读取那个特定要访问的地址，而是读取一个数据块并保存到CPU缓存

下次访问内存数据的时候就会先从CPU缓存开始查找，如果找到就不需要再从内存中取

对于数组来说，存储空间是连续的，所以上次读取时可能附近的元素已经被带到CPU缓存里了

### 链式存储结构

任意的存储空间

数据元素本身(数据域)+直接后继地址(指针域)

头指针必要，头结点不必要

低效随机访问，遍历

高效插入和删除，找的过程依然需要遍历

指针域需要占用空间

#### 单链表

创建方式：

- 头插法，新的结点插在头部
- 尾插法，新的结点插在尾部

整表删除，要释放所有的结点

##### 判断两个单链表相交

根据单链表的定义，一定是一个Y的形状，而不可能是一个X的形状

所以两个单链表相交，尾节点是同一个

##### 判断链表里有环

- 使用两个指针，指针p和指针q。指针p每次都向前走一个位置，指针q每次都从头开始走，一直走到指针p的位置。如果指针p走过的步数和指针q走过的步数不一样，那就存在环。
- 使用两个指针，指针p和指针q。指针p每次走一步，指针q每次走两步。如果某个时刻指针p和指针q相等了，那就存在环。

#### 静态链表

用数组描述链表

数组的元素就是结点，数组的下标就是指针

第一个元素存放数组第一个未被使用的元素的下标

最后一个元素存放数组第一个有效的元素的下标

给没有指针的高级语言设计的一种实现单链表的方法

#### 循环列表(circular linked list)

将单链表中终端结点的指针域由空指针改为指向头结点，使整个单链表形成一个环

#### 双向链表(double linked list)

在单链表的每个结点中设置一个指向其前驱结点的指针域

#### 双向循环列表

双向循环列表同时拥有循环列表和双向链表的特性

### 基于链表实现缓存LRU淘汰算法

每次访问缓存时把访问到的结点取出来插到头部

这样越靠近尾部的节点就是越早访问的

#### 常见的策略缓存淘汰策略

- 先进先出策略 FIFO（First In，First Out）
- 最少使用策略 LFU（Least Frequently Used）
- 最近最少使用策略 LRU（Least Recently Used）

---

| 代码说明   | 代码位置                                    |
| ---------- | ------------------------------------------- |
| 链表       | CYangLi/shu4ju4jie2gou4/lian4biao3.c        |
| 约瑟夫问题 | CYangLi/shu4ju4jie2gou4/yue1se4fu1wen4ti2.c |