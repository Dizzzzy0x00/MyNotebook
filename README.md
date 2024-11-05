# 🐒 Deep Learning with Graphs

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

      <figure><img src=".gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

图的重要特征

* Node Degrees 节点度
  *   应用：侧面反应网络中枢节点

      <figure><img src=".gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>


* Adjacency Matrix 邻接矩阵（图的矩阵表示）
  * 无向图：对称阵；有向图：非对称阵
  * 为什么要把图表示矩阵形式：邻接矩阵  表示了图的全息信息（可以包含**节点、边、权重**），后续矩阵计算
  * 邻接矩阵**缺点**：现实中的大多数图的邻接矩阵都是**稀疏矩阵**
    * **Density of matrix：** $$\frac{E }{N^2}$$
    * 针对这个问题引入：连接列表和邻接列表，只记录连接关系
    *

        <figure><img src=".gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

图的表示学习：自动学习特征，将各个模态的输入转为向量，将节点映射为d维向量（低维 连续 稠密—— distributed vector 分布式向量、task-independent 与下游任务无关）

节点嵌入的目的：向量点乘数值（余弦相似度）反应节点的相似度



## DeepWalk

原文：DeepWalk: Online Learning of Social Representations

{% embed url="https://dl.acm.org/doi/10.1145/2623330.2623732" %}

论文Keyword：Online learning增量学习，实现对图中节点的Embedding Learning 表示学习，输入为图，输出为每个节点对应的k维向量

### 方法 Algorithm：DEEPWALK

* **“Random walk generator” 随机游走生成器**
* **“Update procedure” 迭代优化**

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

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


gama = 10 
walk_length = 5
random_walks = []

for n in tqdm(all_nodes):
  for i in range(game):
    random_walks.append(get_randomwalk(n,walk_length))
    
    
```



## GNN 图神经网络

k-layer GNN 每个节点感受野：k-hop neighborhood



## GCN 图卷积神经网络

由于GNN感受野有限，太高层数的GNN计算图过于复杂，所以映入Neighborhood Aggregation：

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>
