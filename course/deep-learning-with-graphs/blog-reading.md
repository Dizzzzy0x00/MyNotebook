---
description: A Gentle Introduction to Graph Neural Networks
---

# Blog Reading

源博客连接：

{% embed url="https://distill.pub/2021/gnn-intro/" %}



## 图结构

实体+关系

三个属性：顶点、边、整个图

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

图结构信息的表示：

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

图结构数据的问题类型：

* node-level task
  *

      <figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>
* edge-level
  *

      <figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>
* graph-level
  *

      <figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>



机器学习中使用图的挑战

如何表示节点、边、图上下文和连接性，当图节点数巨大时矩阵如何存储、如何让网络学习邻接矩阵排序的同质性：

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>
