# Deep Learning with graph

## Part1 Graphs

### 复杂的图结构

* Arbitrary size and complex topological 任意尺寸输入和复杂的拓扑网络结构
* No fixed node ordering or reference point 图节点无序性
* Dynamic and have multimodal features 多模态和动态图

### Compon ents 图的基本表示

* Objects：nodes 节点，vertices 顶点 **N**
* Interactions：linkes，edges **E**
* System：network，graph **G(N,E)**

#### 图的种类

* Undirected 无向图
* Directed 有向图
*   **Heterogeneous graph 异质图** $$G = （V，E，R，T）$$

    * **节点有不同的种类 V**
    * **不同的边类型 E**
    * **节点类型集合 T(Vi)**
    * **边类型集合 R**
    * eg: T=2时 两种类型的节点 二分图，可以进行展开拆分成U图和V图
    *

    ```
    <figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
    ```

图的重要特征

* Node Degrees 节点度
  *   应用：侧面反应网络中枢节点

      <figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
* Adjacency Matrix 邻接矩阵（图的矩阵表示）
  * 无向图：对称阵；有向图：非对称阵
  * 为什么要把图表示矩阵形式：邻接矩阵 表示了图的全息信息（可以包含**节点、边、权重**），后续矩阵计算
  *   邻接矩阵**缺点**：现实中的大多数图的邻接矩阵都是**稀疏矩阵**

      * **Density of matrix：** $$\frac{E }{N^2}$$
      * 针对这个问题引入：连接列表和邻接列表，只记录连接关系
      *

      ```
      <figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
      ```

图的表示学习：自动学习特征，将各个模态的输入转为向量，将节点映射为d维向量（低维 连续 稠密—— distributed vector 分布式向量、task-independent 与下游任务无关）

节点嵌入的目的：向量点乘数值（余弦相似度）反应节点的相似度

### 图的基本操作

````python
```python
import numpy as np
import pandas as pd
import networkx as nx
import matplotlib.pyplot as plt
 
edges=pd.DataFrame()
edges['sources']=[1,1,1,2,2,3,3,4,4,5,5,5]
#有向边的起点
 
edges['targets']=[2,4,5,3,1,2,5,1,5,1,3,4]
#有向边的终点
 
edges['weights']=[1,1,1,1,1,1,1,2,1,1,1,1]
#各边的权重
 
G=nx.from_pandas_edgelist(
    edges,
    source='sources',
    target='targets',
    edge_attr='weights')
 
 
nx.draw(G, with_labels=True)
plt.show()

#degree
print("degree:{}".format(nx.degree(G)))

#连通分量
print("连通分量: ",list(nx.connected_components(G)))

#图直径
print("diameter：{}".format(nx.diameter(G)))

#中心性
print("度中心性",nx.degree_centrality(G))
print("连接中心性: ",nx.closeness_centrality(G))
print("特征向量中心性: ",nx.eigenvector_centrality(G))
print("中介中心性: ",nx.betweenness_centrality(G))


print("pagerank:",nx.pagerank(G))
print("HITS:",nx.hits(G))


#output:


degree:[(1, 3), (2, 2), (4, 2), (5, 3), (3, 2)]
连通分量:  [{1, 2, 3, 4, 5}]
diameter：2
度中心性 {1: 0.75, 2: 0.5, 4: 0.5, 5: 0.75, 3: 0.5}
连接中心性:  {1: 0.8, 2: 0.6666666666666666, 4: 0.6666666666666666, 5: 0.8, 3: 0.6666666666666666}
特征向量中心性:  {1: 0.5298988890761731, 2: 0.35775191431708964, 4: 0.4271316779596084, 5: 0.5298988890761731, 3: 0.35775191431708964}
中介中心性:  {1: 0.25, 2: 0.08333333333333333, 4: 0.0, 5: 0.25, 3: 0.08333333333333333}
pagerank: {1: 0.24369622576677993, 2: 0.1722562971205864, 4: 0.16809495422526696, 5: 0.24369622576677993, 3: 0.1722562971205864}
HITS: ({1: 0.24059715204600782, 2: 0.16243456471667697, 4: 0.19393656647463045, 5: 0.24059715204600782, 3: 0.16243456471667697}, {1: 0.24059715204600773, 2: 0.16243456471667697, 4: 0.1939365664746304, 5: 0.24059715204600784, 3: 0.162434564716677})
````

<figure><img src="../../.gitbook/assets/bf3165fc9fcaed7b268dc3605317f99.png" alt=""><figcaption></figcaption></figure>

## Graph embedding 图嵌入算法

### DeepWalk

原文：DeepWalk: Online Learning of Social Representations

{% embed url="https://dl.acm.org/doi/10.1145/2623330.2623732" %}

论文Keyword：Online learning增量学习，实现对图中节点的Embedding Learning 表示学习，输入为图，输出为每个节点对应的k维向量

### 方法 Algorithm：DEEPWALK

* **“Random walk generator” 随机游走生成器**
* **“Update procedure” 迭代优化**

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

语言模型推广：中心词预测周围词，通过随机游走路径来进行语言建模——用前i-1个节点预测第i个节点 $$Pr(v_i | (\Phi(v_1),\Phi(v_2),...,\Phi(v_{i-1}))),这里\Phi(v_k)表示节点k的嵌入向量$$,但是在节点数量很多时引入连乘的条件概率会导致太小而不可行，所以更改为优化损失：

$$
minimize -log Pr(\frac{v_{i-w},...,v_{i+w}}{v_i} | \Phi(v_i))
$$

重点：

* 损失函数的计算 $$J(\Phi) = -logPr(u_k|\Phi(v_j))$$$$= \prod_{j=i-w,j=i}^{i+w}Pr(v_j|\Phi(v_i))$$

### 代码实战——维基百科词条DeepWalk图嵌入

```python
import networkx as nx

import pandas as pd
import numpy as np

import random
from tqdm import tqdm
  
df.head()

#构建无向图：

G = nx.from_pandas_edgelist(df,"source","target",edges_attr=True,create_using=nx.Graph())
print(len(G))

def get_randomwalk(node,path_length):
  random_walk = [node] //起始节点
  for i in range(path_length-1):
    temp = list(G.neighbore(node)) //汇总当前节点的所有邻接节点
    temp = list(set(temp)-set(random_walk))
    if len(temp)== G:
      break
    random_node = random.choice(temp) //随机选择下一个节点
    random_walk.append(random_node)
    node = random_node
    
  return random_walk


gama = 10 #迭代次数
#（gama并不是越大越好，到达一定大小值以后gama增加对嵌入效果影响越来越小）
walk_length = 5
random_walks = []

for n in tqdm(all_nodes):
  for i in range(game):
    random_walks.append(get_randomwalk(n,walk_length))
    
    
```

### LINE：大规模信息网络嵌入

原文：large-scale Information Network Embedding

{% embed url="https://arxiv.org/abs/1503.03578" %}

相较于DeepWalk，Line在大规模的图上有更好的效果，在有向图上可以使用

核心思想是考虑节点的两阶信息：

* 一阶：局部的结构信息
* 二阶：节点的邻居，共享邻居的节点可能是相似的

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

**一阶邻近度**

一阶邻近度是指网络中顶点之间的局部成对邻近度。 为了模拟一阶邻近度，对于每个无向边`(i, j)`，我们定义顶点`v[i]`和`v[j]`之间的联合概率，如下所示：

$$
p_1(v_i,v_j)=\frac{1}{1+exp(-\vec u_i.\vec u_j)}                  (1)
$$

其中 $$\vec u_i \in R^d$$是顶点`v[i]`的低维矢量表示。公式（1）定义空间`V×V`上的分布`p`，其经验概率可定义为 $$\hat p_1(i, j) = \frac {w_{i,j}}{W}$$其中`W = Σw[ij], (i, j) ∈ E`。 为了保留一阶邻近度，一种直接的方法是最小化以下目标函数：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS83LzcvMTZiY2MxNzUxYmNhMDEyOA) (2)

其中`d(·,·)`是两个分布之间的距离。 我们选择最小化两个概率分布的 KL 散度。 用 KL 散度代替`d(·,·)`并省略一些常数，我们得到：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS83LzcvMTZiY2MxNzUxOTJmM2JiZA) (3)

请注意，一阶邻近度仅适用于无向图，而不适用于有向图。 通过找到最小化公式（3）中的目标的`{u[i]}, i = 1 .. |V|`，我们可以表示`d`维空间中的每个顶点。

### Node2Vec

几个概念

* 同质性homophily：节点与其周围的节点的embedding应该是相近的
* 架构等价性structural equivalence：结构相似的节点的embedding应该是相近的
* 广度优先搜索BFS
* 深度优先搜索DFS

采用有策略的游走策略

从t节点出发，从节点v跳转到下一个节点x的概率为：

$$
\pi_{vx}=\alpha_{pq}(t,x)\cdot\omega_{vx}
$$

$$\alpha_{pq}$$表示走向下一个节点x的概率如下：

$$
\alpha_{pq}(t,x)=\begin{cases}
\frac{1}{p} (\text{if} d_tx = 0)\\
1(\text{if} d_tx = 1)\\
\frac{1}{q} (\text{if} d_tx = 2)\\
\end{cases}
$$

d表示距离，

* 如果两个节点为相同的节点， $$d_{tx}=0$$
* 如果两个节点直接相连， $$d_{tx}=1$$
* 如果两个节点不相连， $$d_{tx}=2$$

pq参数控制游走的策略，

* 参数p被称为返回参数（return parameter），p越小，节点返回t的概率越高，[图的遍历](https://zhida.zhihu.com/search?content_id=226129772\&content_type=Article\&match_order=1\&q=%E5%9B%BE%E7%9A%84%E9%81%8D%E5%8E%86\&zhida_source=entity)越倾向于BFS，越趋向于表示图的结构性。
* 参数q被称为进出参数（in-out parameter），q越小，遍历到远处节点的概率越高，图的遍历越倾向于DFS，越趋向于表示图的同质性。

<figure><img src="../../.gitbook/assets/image (88).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (89).png" alt=""><figcaption></figcaption></figure>

### Struc2Vec

Node2Vec的一个缺陷是无法学习到充分的结构相似性，因为random walk的步数有限

<figure><img src="../../.gitbook/assets/image (90).png" alt=""><figcaption></figcaption></figure>

Struc2Vec捕捉网络中不相连但是结构相似的节点信息，忽略节点和边的属性以及它们在网络中的位置来评估节点间的结构相似性，仅考虑节点的局部结构。两个节点的结构相似度的直观评判标准是：**如果两个节点的度相同，它们在结构上是相似的；如果两个节点的邻居的度也相同，它们的结构相似度就更高了**。

* **计算结构相似性**

在khop邻居的u和v的相似性递推公式：

$$
\begin{cases} f_k(u,v)=f_{k-1}(u,v)+g(sR_k(u),sR_k(v))\\f_{-1}(u,v)=0\\\end{cases}
$$

定义Rk(u)表示在khop的顶点集合，S(s)表示集合S度的有序序列，g(D1,D2)表示D1与D2之间的距离,可以用动态时间规整算法进行计算：

#### DWT动态时间规整算法

（Dynamic Time Warping, DWT）衡量两个序列的相似性，DWT通过动态规划的方法找到两个时间序列之间最优对齐路径，使得序列之间的距离（通常为欧几里得距离或其他相似度度量）最小。

给定两个时间序列：

* $$A=[a1,a2,...,an]A = [a_1, a_2, ..., a_n]A=[a1​,a2​,...,an​]$$
* $$B=[b1,b2,...,bm]B = [b_1, b_2, ..., b_m]B=[b1​,b2​,...,bm​]$$

构造一个 $$n×mn \times mn×m$$ 的累积距离矩阵 $$D$$，其中 $$D(i,j)$$表示从序列起点到位置 $$(i,j)$$的最小累积距离。

递归关系如下：

$$
D(i,j)=d(a_i,b_j)+min(D(i−1,j),D(i,j−1),D(i−1,j−1))
$$

* $$d(a_i, b_j)$$：序列 AAA 和 BBB 在位置 i,ji, ji,j 的局部距离（如欧几里得距离）。
* 边界条件：起点 $$D(0, 0) = d(a_1, b_1)$$。

最优路径对应的累积距离D(n, m) 即为两个序列的动态时间规整距离。

* **构造多层图**\
  根据节点之间的结构相似性，将节点分配到不同的层，每层表示不同的结构相似性尺度。
  * 在较低的层，节点的结构差异较小。
  * 在较高的层，节点之间的结构相似性更加宽松。

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* **构建边权重**\
  在每一层内和层间连接节点，边的权重反映节点的结构相似性。
  * 在层内：相似的节点通过边连接，权重越高表示结构越相似。
  * 在层间：相同节点的不同层实例通过边连接，形成一种“跨层”的信息通道。

## GNN 图神经网络

k-layer GNN 每个节点感受野：k-hop neighborhood

## GCN 图卷积神经网络

### 拉普拉斯矩阵与图傅里叶变换

图拉普拉斯矩阵是图信号处理与图神经网络中的核心工具之一。其构造反映了图的结构特征，广泛用于图的平滑、谱分析与卷积运算等。

#### 归一化图拉普拉斯矩阵

假设图是一个无向图，记该图为 $$G = (V, E)$$，其中 $$V$$ 是节点集合，$$E$$ 是边集合。设邻接矩阵为 $$A \in \mathbb{R}^{n \times n}$$，度矩阵为 $$D \in \mathbb{R}^{n \times n}$$，是一个对角矩阵，其对角线上第 $$i$$ 个元素为节点 $$i$$ 的度数 $$D_{ii} = \sum_j A_{ij}$$。

**归一化图拉普拉斯矩阵定义为：**

$$
L = I - D^{-\frac{1}{2}} A D^{-\frac{1}{2}}
$$

其中：

* $$I$$ 是单位矩阵。
* $$L$$ 是对称归一化图拉普拉斯矩阵。

#### 谱分解性质

归一化图拉普拉斯矩阵 $$L$$ 是一个实对称半正定矩阵，因此它可以被特征值分解（谱分解）为：

$$
L = U \Lambda U^\top
$$

其中：

* $$U \in \mathbb{R}^{n \times n}$$ 是由 $$L$$ 的**特征向量按特征值升序排列构成的正交矩阵**（$$U^\top U = I$$），
* $$\Lambda \in \mathbb{R}^{n \times n}$$ 是对角特征值矩阵。

这意味着特征向量 $$U$$ 构成一个正交基空间，即：

$$
U^\top U = I
$$

#### 图傅里叶变换与逆变换

图信号是定义在图节点上的函数。设图上每个节点有一个信号值，整个图的信号可以表示为一个向量：

* $$x \in \mathbb{R}^n$$：表示所有节点的信号。
* $$x_i$$：表示第 $$i$$ 个节点的信号。

**图傅里叶变换**是将图信号投影到图的拉普拉斯特征空间中，定义为：

$$
\hat{x} = U^\top x
$$

其中 $$\hat{x}$$ 是图信号在频域上的表示（傅里叶变换结果信号）。

**逆图傅里叶变换**则为：

$$
x = U \hat{x}
$$

因此，图傅里叶变换和其逆变换具有如下关系：

* 傅里叶变换：$$\hat{x} = U^\top x$$
* 逆傅里叶变换：$$x = U \hat{x}$$

#### 傅里叶变换的几何含义

从谱角度看，特征向量矩阵 $$U$$ 的列构成了**图拉普拉斯算子的“频率基”**，即：

* $$U$$ 的低频部分对应于图中变化缓慢的模式（平滑区域），
* 高频部分则对应于图中变化剧烈的模式（边界、异常点）。

图傅里叶变换实质上将图信号 $$x$$ 投影到由拉普拉斯矩阵特征向量构成的正交空间中：

$$
x = \sum_{i=1}^n \hat{x}_i u_i
$$

其中：

* $$u_i$$ 是第 $$i$$ 个特征向量，
* $$\hat{x}_i$$ 是信号 $$x$$ 在基向量 $$u_i$$ 上的投影系数。

因此，**原始输入信号可以被表示为特征空间的线性组合**，这正是谱图卷积等方法的基础。

***

由于GNN感受野有限，太高层数的GNN计算图过于复杂，所以映入Neighborhood Aggregation：

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
