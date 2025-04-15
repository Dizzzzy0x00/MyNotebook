---
description: A Gentle Introduction to Graph Neural Networks
---

# GNN Blog Reading

源博客连接：

{% embed url="https://distill.pub/2021/gnn-intro/" %}



## 图结构

实体+关系

三个属性：顶点、边、整个图

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

图结构信息的表示：

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

图结构数据的问题类型：

* node-level task
  *

      <figure><img src="../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>
* edge-level
  *

      <figure><img src="../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>
* graph-level
  *

      <figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>



机器学习中使用图的挑战

如何表示节点、边、图上下文和连接性，当图节点数巨大时矩阵如何存储、如何让网络学习邻接矩阵排序的同质性：

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption><p>所有邻接矩阵表示的图都是同质图，仅仅交换了节点顺序</p></figcaption></figure>

## Graph Neural Network（GNN）

GNN输入是graph，输出也是graph，并且网络不会改变节点之间的连接性

pooling：

* 有边特征向量但是没有节点特征向量，需要对节点进行预测分类时，使用pooling function将边特征向量聚合为节点向量然后做出预测
* **没有整个图的全局向量，但是有顶点特征向量，将所有顶点的向量进行聚合，经过全局输出层得到全局向量**

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

### 图部分之间的信息传递 “Passing message”



<figure><img src="../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

如何表示全局信息：一种解决方式是使用master node（连接网络中的所有节点和边）来表示整个网络的全局信息 $$U_n$$

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>





