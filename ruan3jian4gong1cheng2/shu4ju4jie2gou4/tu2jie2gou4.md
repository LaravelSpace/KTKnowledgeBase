## 图结构

图（Graph）是由顶点的有穷非空集合和顶点之间边的集合组成，通常表示为：G(V,E)，其中，G表示一个图，V是图G中顶点的集合，E是图G中边的集合。线性表中的数据元素叫元素，树中的数据元素叫结点，图中的数据元素叫顶点。图结构中，不允许没有顶点，集合V是有穷非空集合。图结构中，任意两个顶点之间都有可能有关系，顶点之间的逻辑关系用边来表示。

### 相关定义

- 无向边：顶点vi到vj之间的边没有方向，用无序偶对（vi,vj）表示。
- 无向图：图中任意两个顶点之间的边都是无向边。
- 有向边：从顶点vi到vj之间的边有方向，也称为弧，用有序偶对<vi,vj>表示，vi称为弧尾，vj称为弧头。
- 有向图：图中任意两个顶点之间的边都是有向边。
- 简单图：在图中不存在顶点到其自身的边，且同一条边不重复出现。
- 无向完全图：无向图中，任意两个顶点之间都存在边。
- 有向完全图：有向图中，任意两个顶点之间都存在方向互为相反的两条弧。
- 有很少条边或弧的图称为稀疏图，反之就是稠密图，这是个相对概念。
- 权：与图的边或弧相关的数。
- 网：带权的图。
- 子图：假设有两个图G(V,E)，G(V1,E1)，如果$V1 \subseteq V2且E1 \subseteq E$，则G1是G的子图。

### 顶点与边的关系

对于无向图G=(V,E)，如果边$(V,V1) \in E$，则称顶点V和V1互为邻接点，V和V1相邻接。边（V,V1）依附于顶点V和V1，或者边（V,V1）和顶点V和V1相关联。顶点的度是和顶点相关联的边的数目。

对于有向图G=(V,E)，如果弧$<V,V1> \in E$，则称顶点V邻接到顶点V1，顶点V1邻接自顶点V。弧<V,V1>和顶点V和V1相关联。顶点的入度是以顶点为头的弧的数目，顶点的出度是以顶点为尾的弧的数目。

路径的长度是路径上的边或弧的数目。

第一个顶点到最后一个顶点，如果最后一个顶点和第一个顶点相同，则称这种路径为回路或者环。序列中顶点不重复出现的路径称为简单路径。除了第一个顶点和最后一个顶点之外，其余顶点不重复出现的回路，成为简单回路或者简单环。

### 连通图相关

在无向图G中，如果从顶点V到顶点V1有路径，则称V和V1是连通的，如果对于图中任意两个顶点，都是连通的，则称G是连通图。

无向图的极大连通子图称为连通分量。连通分量：1、要是子图；2、子图要是连通的；3、连通子图含有极大顶点数；4、具有极大顶点数的连通子图包含依附于这些顶点的所有边。

在有向图G中，如果对于每一对V1，V2，从V1到V2和从V2到V1都存在路径，则称G是强连通图。有向图中的极大强连通子图称做有向图的强连通分量。

连通图的生成树是一个极小的连通子图，它含有图中全部的n个顶点，但只有足以构成一棵树的n-1条边。

如果一个有向图恰有一个顶点的入度为0，其余顶点的入度均为1，则是一颗有向树。

一个有向图生成的森林由若干棵有向树组成，含有图中的全部顶点，但只有足以构成若干棵不相交的有向树的弧。
