---
description: 现有研究方法调研（仅关注支持跨架构的研究）
---

# BCSD1-Awesome-Binary-Similarity

## CI-Detector,2024,CCF-A

{% embed url="https://github.com/island255/cross-inlining_binary_function_similarity" %}

{% embed url="https://dl.acm.org/doi/10.1145/3597503.3639080" %}

提出了一个基于模式的模型 CI-Detector，用于交叉内联匹配。CI-Detector 使用**带属性 的CFG（attributed CFG）** 来表示二元函数的语义，使用 **GNN** 将二元函数嵌入到向量中。CI-Detector 分别针对这三种交叉内联模式训练一个模型。文章使用的数据集跨架构（作者构建一个交叉内联数据集，使用 9 个编译器，4 种优化器，将 51 个项目编译到 6 个架构——2 个内联标志中，从而得到两个数据集，每个数据集都有 216 种组合）文章核心关注的是对交叉内联匹配的相似性检测。



## HermesSim,2024,CCF-A

{% embed url="https://github.com/NSSL-SJTU/HermesSim" %}

{% embed url="https://www.usenix.org/conference/usenixsecurity24/presentation/he-haojie" %}

之前的大多数BCSD研究方式可以归纳为两大流派：

* 基于指令流的（instruction streams）：使用NLP对二进制代码进行分析
* 基于控制流图的_**（control flow graphs ，CFGs）**_：使用图机器学习进行分析

作者认为基于指令流的方法将二进制代码视为自然语言，忽略了明确定义的语义结构。基于 CFG 的方法仅利用控制流结构，而忽略了代码的其他基本方面。

文章作者的见解是与自然语言不同，二进制代码具有明确定义的语义结构，包括指令内结构、指令间关系（例如，def-use、分支）和隐式约定（例如调用约定）。所以作者尝试构建一种新的图_**（semantics-oriented graph ，SOG）**_来表达二进制程序完整语义结构，递交给深度神经网络。此外提出了一个轻量级的多头 softmax 聚合器，以有效、高效地融合二进制代码的多个方面。

<figure><img src="../.gitbook/assets/image (84).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (85).png" alt=""><figcaption></figcaption></figure>
