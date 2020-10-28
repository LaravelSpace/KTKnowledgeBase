## 架构演进

架构设计不是一蹴而就的，是一个不断贴合实际场景并不断演进的过程。

### 基本概念

分布式：系统中的多个模块在不同服务器上部署，即可称为分布式系统。

高可用：系统中部分节点失效时，其他节点能够接替它继续提供服务。

集群：一个特定领域的软件部署在多台服务器上并作为一个整体提供一类服务，这个整体称为集群。在常见的集群中，客户端往往能够连接任意一个节点获得服务，并且当集群中一个节点掉线时，其他节点往往能够自动的接替它继续提供服务，这时候说明集群具有高可用性。

负载均衡：请求发送到系统时，通过某些方式把请求均匀分发到多个节点上，使系统中每个节点能够均匀的处理请求负载，则可认为系统是负载均衡的。

正向代理：系统内部要访问外部网络时，统一通过一个代理服务器把请求转发出去，在外部网络看来就是代理服务器发起的访问，此时代理服务器实现的是正向代理。正向代理是代理服务器代替系统内部来访问外部网络的过程。

反向代理：当外部请求进入系统时，代理服务器把该请求转发到系统中的某台服务器上，对外部请求来说，与之交互的只有代理服务器，此时代理服务器实现的是反向代理。反向代理是外部请求访问系统时通过代理服务器转发到内部服务器的过程。

### 架构演进

文章以淘宝作为例子，介绍从一百个并发到千万级并发情况下服务端的架构的演进过程。

#### 单机架构

一个Web应用在最初时，应用数量和用户数量都比较少，可以把服务端应用和数据库应用都部署在一台服务器上。

#### 第一次演进：Tomcat与数据库分开部署

随着用户数量的增加，服务端应用和数据库应用开始竞争资源，这个时候就需要将两者分开到各自独立的服务器部署。

#### 第二次演进：引入本地缓存和分布式缓存

随着用户数量进一步的增加，数据库并发请求成为瓶颈。但是在这个阶段，数据库读请求的数量和数据库写请求的数量是不成比例的，通常是读请求数量要远大于写请求数量。这个阶段就需要引入缓存，缓存能把大多数的数据库读请求拦截住，从而降低数据库的压力。

这里涉及到本地缓存、分布式缓存、缓存一致性、缓存穿透、缓存雪崩、热点数据集中失效等问题。

#### 第三次演进：引入反向代理实现负载均衡

服务端应用压力进一步上升，这个时候一台服务器已经无法满足大量的并发请求，所以就需要在多台服务器上部署服务端应用。

#### 第四次演进：数据库读写分离



#### 第五次演进：数据库按业务分库



#### 第六次演进：把大表拆分为小表



#### 第七次演进：使用LVS或F5来使多个Nginx负载均衡



#### 第八次演进：通过DNS轮询实现机房间的负载均衡



#### 第九次演进：引入NoSQL数据库和搜索引擎等技术



#### 第十次演进：大应用拆分为小应用





#### 第十一次演进：复用的功能抽离成微服务



#### 第十二次演进：引入企业服务总线ESB屏蔽服务接口的访问差异



#### 第十三次演进：引入容器化技术实现运行环境隔离与动态服务管理



#### 第十四次演进：以云平台承载系统