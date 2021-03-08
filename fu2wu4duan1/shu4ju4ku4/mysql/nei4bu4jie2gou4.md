## MySQL内部结构

MySQL由连接池、SQL接口、解析器、优化器、缓存、存储引擎等组成，可以分为三层，即MySQL Server层、存储引擎层和文件系统层。MySQL Server层又包括连接层和SQL层。下图是官方文档中MySQL的基础架构图：

![](E:\GongZuoQu\ZhiShiKu\tu2pian4\fu2wu4duan1\shu4ju4ku4\mysql\mysql0nei4bu4jie2gou4.png)

Connectors不属于以上任何一层，可以将Connectors理解为各种客户端、应用服务，主要指的是不同语言与SQL的交互。

Connection pool为连接层，Management Services & Utilities...Caches & Buffers为SQL层，Pluggable Storage Engines为存储引擎层，Filesystem、Files&Logs为文件系统层。

### 连接层

应用程序通过接口（如ODBC、JDBC）来连接MySQL，最先连接处理的是连接层。连接层包括通信协议、线程处理、用户名密码认证3部分。

- 通信协议负责检测客户端版本是否兼容MySQL服务端。
- 线程处理是指每一个连接请求都会分配一个对应的线程，相当于一条SQL对应一个线程，一个线程对应一个逻辑CPU，在多个逻辑CPU之间进行切换。
- 密码认证用来验证用户创建的账号、密码，以及host主机授权是否可以连接到MySQL服务器。

Connection Pool（连接池）属于连接层。由于每次建立连接都需要消耗很多时间，连接池的作用就是将用户连接、用户名、密码、权限校验、线程处理等需要缓存的需求缓存下来，下次可以直接用已经建立好的连接，提升服务器性能。

### SQL层

SQL层是MySQL的核心，MySQL的核心服务都是在这层实现的。主要包含权限判断、查询缓存、解析器、预处理、查询优化器、缓存和执行计划。

- 权限判断可以审核用户有没有访问某个库、某个表，或者表里某行数据的权限。
- 查询缓存通过Query Cache进行操作，如果数据在Query Cache中，则直接返回结果给客户端，不必再进行查询解析、优化和执行等过程。
- 查询解析器针对SQL语句进行解析，判断语法是否正确。
- 预处理器对解析器无法解析的语义进行处理。
- 查询优化器对SQL进行改写和相应的优化，并生成最优的执行计划，就可以调用程序的API接口，通过存储引擎层访问数据。


Management Services & Utilities、SQL Interface、Parser、Optimizer和Caches & Buffers属于SQL层，详细说明如下表所示。

| 名称                            | 说明                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| Management Services & Utilities | MySQL的系统管理和控制工具，包括备份恢复、MySQL复制、集群等。 |
| SQL Interface（SQL接口）        | 用来接收用户的SQL命令，返回用户需要查询的结果。例如SELECTFROM就是调用SQL Interface。 |
| Parser（查询解析器）            | 在SQL命令传递到解析器的时候会被解析器验证和解析，以便MySQL优化器可以识别的数据结构或返回SQL语句的错误。 |
| Optimizer（查询优化器）         | SQL语句在查询之前会使用查询优化器对查询进行优化，同时验证用户是否有权限进行查询，缓存中是否有可用的最新数据。它使用选取-投影-连接策略进行查询。例如`SELECT id,name FROM student WHERE gender="女";`语句中，SELECT查询先根据WHERE语句进行选取，而不是将表全部查询出来以后再进行gender过滤。SELECT查询先根据id和name进行属性投影，而不是将属性全部取出以后再进行过滤，将这两个查询条件连接起来生成最终查询结果。 |
| Caches & Buffers（查询缓存）    | 如果查询缓存有命中的查询结果，查询语句就可以直接去查询缓存中取数据。这个缓存机制是由一系列小缓存组成的，比如表缓存、记录缓存、key缓存、权限缓存等。 |

### 存储引擎层

Pluggable Storage Engines属于存储引擎层。存储引擎层是MySQL数据库区别于其他数据库最核心的一点，也是MySQL最具特色的一个地方。主要负责MySQL中数据的存储和提取。

因为在关系数据库中，数据的存储是以表的形式存储的，所以存储引擎也可以称为表类型（即存储和操作此表的类型）。

### 文件系统层

文件系统层主要是将数据库的数据存储在操作系统的文件系统之上，并完成与存储引擎的交互。