## 说明文档

### 项目说明

项目名称：知识库。

注意：知识库的内容来源比较广泛，存在大量未成体系的零散的部分。

### 目录结构

目录结构的设计原则是清晰，不要为难自己用不擅长的方式去命名。目录结构和文件一般都拼音命名（例如：软件工程=>ruan3jian4gong1cheng2），也可以是外文（驼峰或蛇形）。拼音命名和外文混用时，外文单词最后要加一个0。如果要对文件分组或排序，可以使用前缀，比如：[21文件名]、[22文件名]。目录结构建议控制在3-4层，如果3-4个关键字还找不到目标，那么就需要改进改进了。

下面是项目的目录结构：

- cao1zuo4xi4tong3：操作系统
- - homestead
  - - laravel：使用Homestead搭建Laravel开发环境
    - redis：在Homestead中使用Redis
    - swoole：在Homestead中安装Swoole扩展
  - linux
  - - shell
    - - chang2yong4ming4ling4：常用shell命令
      - ji4hua4ren4wu4：计划任务
  - windows
  - - chang2yong4cao1zuo4：Windows常用操作
- FuWuDuan：服务端
- GeekBang：极客时间（等待整理）
- Image：图片目录
- JiKeShiJian：极客时间
- LeetCode：LeetCode算法题目
- RuanJianGongCheng：软件工程相关知识  
- [ruan3jian4gong1cheng2]：软件工程
- - [suan4fa3]：算法
  - - [dong4tai4gui1hua4]：动态规划
    - - dong4tai4gui1hua4：动态规划
      - guo2wang2he2jin1kuang4：国王和金矿
      - pa2tai2jie1：爬台阶
      - zui4chang2gong1gong4zi3xu4lie4：最长公共子序列
      - zui4xiao3lu4jing4：最小路径
  - git：git常用命令
  - jia4gou4yan3jin4：架构演进
- ShuJi：书籍
- TuPian：图片目录

### 文件内容

文件标题建议使用二级标题（\#\#），不建议使用一级标题（太大了）。

文件头部和文件主体之间、文件主体和参考来源之间使用分割线（---）分开。

### 参考来源

参考来源用于标识文件内容的来源。如果是独立的内容，参考来源放置于文件的最后。如果是成体系的一系列内容，参考来源放置于目录结构说明文档。

- 如果完全是自己整理的，那就不需要标明来源。
- 如果是来自网络的或者来自出版物的，要标明出处。
- 最后推荐有能力的朋友们去购买正版的软件和付费课程。

### 代码位置

知识库内部分文件会出现大段的代码，要求标注代码片段是哪一类代码。

- 大部分C代码的源文件都放置于CYangLi项目。
- 大部分PHP代码的源文件都放置于PHPYangLi项目。
- 大部分Golang代码的源文件都放置于GoYangLi项目。
- 当然还有可能是独立的项目，标记一下来源就行了。

### 图片

图片以：**[文件名]_img[编号]**的格式命名。放置于TuPian文件夹，暂时不提交至GitHub。

