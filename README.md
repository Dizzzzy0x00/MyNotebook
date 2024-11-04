# Deep Learning with Graphs

## Part1 Graphs



### 复杂的图结构

* Arbitrary size and complex topological 任意尺寸输入和复杂的拓扑网络结构
* No fixed node ordering or reference point 图节点无序性
* Dynamic and have multimodal features 多模态和动态图



### Compon ents 图的基本表示

* Objects：nodes 节点，vertices 顶点         **N**
* Interactions：linkes，edges                       **E**
* System：network，graph                           **G(N,E)**



#### 图的种类

* Undirected 无向图
* Directed 有向图
* **Heterogeneous graph 异质图**   $$G = （V，E，R，T）$$
  * **节点有不同的种类   V**
  * **不同的边类型           E**
  * **节点类型集合           T(Vi)**
  * **边类型集合               R**
  * eg: T=2时 两种类型的节点 二分图，可以进行展开拆分成U图和V图
  *

      <figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

图的重要特征

* Node Degrees 节点度
  *   应用：侧面反应网络中枢节点

      <figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>


* Adjacency Matrix 邻接矩阵（图的矩阵表示）
  * 无向图：对称阵；有向图：非对称阵
  * 为什么要把图表示矩阵形式：邻接矩阵  表示了图的全息信息（可以包含**节点、边、权重**），后续矩阵计算
  * 邻接矩阵**缺点**：现实中的大多数图的邻接矩阵都是**稀疏矩阵**
    * **Density of matrix：** $$\frac{E }{N^2}$$
    * 针对这个问题引入：连接列表和邻接列表，只记录连接关系
    *

        <figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

图的表示学习：自动学习特征，将各个模态的输入转为向量，将节点映射为d维向量（低维 连续 稠密—— distributed vector 分布式向量、task-independent 与下游任务无关）
