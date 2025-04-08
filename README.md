---
icon: github-alt
cover: .gitbook/assets/微信图片_20250408134702.jpg
coverY: 145
---

# Deep Learning

## RNN

Recurrent Neural Network，循环神经网络，用于处理**序列数据**的神经网络。RNN具有内部记忆功能，可以在序列中的当前步骤依赖于前一步骤的信息。这使得RNN特别适合于处理诸如时间序列、语言文本、语音识别等需要考虑上下文的任务。

### 数学原理

**1. 基本结构**

RNN 的基本单元由一个循环结构组成，它允许信息在网络的时间步骤之间传递。假设我们有一个输入序列 $$({x^{(1)}, x^{(2)}, \dots, x^{(T)}})$$，RNN 通过以下公式来更新其隐藏状态和输出：

*   **隐藏状态更新：**

    \
    $$h^{(t)} = f\bigl(W_{hh} h^{(t-1)} + W_{xh} x^{(t)} + b_h\bigr)$$\
    \
    其中：

    * $$h^{(t)}$$是时间步 (t) 的隐藏状态。
    * $$W_{hh}$$是隐藏状态到隐藏状态的权重矩阵，维度为 $$\mathbb{R}^{H\times H}$$。
    * $$W_{xh}$$是输入到隐藏状态的权重矩阵，维度为 $$\mathbb{R}^{D\times H}$$。
    * $$b_h$$ 是隐藏层的偏置项，维度为 $$\mathbb{R}^{H}$$。
    * $$f(x)$$是激活函数通常是 $$\tanh$$或 $$ReLU$$）。
*   **输出计算：**

    \
    $$y^{(t)} = g\bigl(W_{hy} h^{(t)} + b_y\bigr)$$

    其中：

    * $$y^{(t)}$$ 是时间步 t 的输出。
      * $$W_{hy}$$ 是隐藏状态到输出的权重矩阵，维度为 $$\mathbb{R}^{H\times O}$$。
      * $$b_y$$是输出层的偏置项，维度为 $$\mathbb{R}^{O}$$。
      * $$g$$是输出的激活函数（根据任务不同，可能是 softmax、sigmoid 等）。

***

**2. 前向传播过程**

1.  **初始化隐藏状态：**

    \
    $$h^{(0)} = \mathbf{0} \quad\text{或随机初始化}$$

    1.  **通过时间步更新隐藏状态和输出：**

        \
        对于每个时间步 $$t=1,2,\dots,T$$：

        *   计算隐藏状态：

            \
            $$h^{(t)} = f\bigl(W_{hh} h^{(t-1)} + W_{xh} x^{(t)} + b_h\bigr)$$
        *   计算输出：

            \
            $$y^{(t)} = g\bigl(W_{hy} h^{(t)} + b_y\bigr)$$

***

**3. 损失函数和反向传播过程**

*   **总损失：**

    \
    $$\mathcal{L} = \sum_{t=1}^{T} \ell\bigl(y^{(t)},\, \hat{y}^{(t)}\bigr)$$

    \
    其中：
* $$\ell$$ 是每个时间步的损失函数（如交叉熵损失）。\\
* $$\hat{y}^{(t)}$$ 是时间步 (t) 的真实值。
* **BPTT（Backpropagation Through Time）：**
  1.  **初始化梯度：**

      $$
      \frac{\partial \mathcal{L}}{\partial W_{hh}} = 0,\quad
        \frac{\partial \mathcal{L}}{\partial W_{hy}} = 0,\quad
        \frac{\partial \mathcal{L}}{\partial b_h} = 0,\quad
        \frac{\partial \mathcal{L}}{\partial b_y} = 0.\
      $$
  2. **反向传播误差（从 (t=T) 到 (t=1)）：**
     *   输出层梯度：

         $$
         z^{(t)} = W_{hy} h^{(t)} + b_y.\
         $$
     *   隐藏层梯度：

         \
         $$\delta_h^{(t)} = \bigl(W_{hy}^\top \delta_y^{(t)} + W_{hh}^\top \delta_h^{(t+1)}\bigr) \odot f'\bigl(a^{(t)}\bigr),$$

         \
         其中 $$a^{(t)} = W_{hh} h^{(t-1)} + W_{xh} x^{(t)} + b_h$$。
     *   累计权重梯度：

         $$
         \begin{aligned}
           \frac{\partial \mathcal{L}}{\partial W_{hy}} &\mathrel{+}= \delta_y^{(t)}\,h^{(t)\top},\\
           \frac{\partial \mathcal{L}}{\partial b_y}  &\mathrel{+}= \delta_y^{(t)},\\
           \frac{\partial \mathcal{L}}{\partial W_{hh}} &\mathrel{+}= \delta_h^{(t)}\,h^{(t-1)\top},\\
           \frac{\partial \mathcal{L}}{\partial W_{xh}} &\mathrel{+}= \delta_h^{(t)}\,x^{(t)\top},\\
           \frac{\partial \mathcal{L}}{\partial b_h}  &\mathrel{+}= \delta_h^{(t)}.
         \end{aligned}
         \
         $$
  3.  **参数更新（梯度下降）：**

      $$
      \begin{aligned}
        W_{xh} &\leftarrow W_{xh} - \eta \frac{\partial \mathcal{L}}{\partial W_{xh}},\\
        W_{hh} &\leftarrow W_{hh} - \eta \frac{\partial \mathcal{L}}{\partial W_{hh}},\\
        W_{hy} &\leftarrow W_{hy} - \eta \frac{\partial \mathcal{L}}{\partial W_{hy}},\\
        b_h    &\leftarrow b_h    - \eta \frac{\partial \mathcal{L}}{\partial b_h},\\
        b_y    &\leftarrow b_y    - \eta \frac{\partial \mathcal{L}}{\partial b_y}.
      \end{aligned}
      $$

RNN 通过循环结构捕捉序列中的依赖关系，通过前向传播计算隐藏状态和输出，并通过时间反向传播来更新权重，从而在处理序列数据时有效利用上下文信息。

在优化RNN模型时，可以考虑以下技术：

1. **正则化**：使用Dropout层来防止过拟合。
2. **不同的RNN变体**：尝试LSTM或GRU，这些变体在处理长序列时表现更好。
3. **调整超参数**：通过网格搜索或随机搜索调整学习率、批量大小、隐藏单元数等超参数。
4. **数据增强**：扩充训练数据集，增加模型的泛化能力。

### 代码实现

简单的实现代码：

```python
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import SimpleRNN, Dense, Dropout

model = Sequential()
model.add(SimpleRNN(units=50, return_sequences=True, input_shape=(X.shape[1], 1)))
model.add(Dropout(0.2))
model.add(SimpleRNN(units=50))
model.add(Dropout(0.2))
model.add(Dense(1))

model.compile(optimizer='adam', loss='mean_squared_error')

# 训练模型
history = model.fit(X, y, epochs=50, batch_size=32, validation_split=0.2, verbose=1)

# 可视化训练过程
plt.figure(figsize=(14, 7))
plt.plot(history.history['loss'], label='Training Loss')
plt.plot(history.history['val_loss'], label='Validation Loss')
plt.title('Model Loss During Training')
plt.xlabel('Epoch')
plt.ylabel('Loss')
plt.legend()
plt.show()
```

## LSTM

Long Short-Term Memory，长短期记忆网络，是循环神经网络（RNN）的特殊形式，通过引入遗忘门、输入门和输出门来控制信息的流动，从而允许网络更有效地学习和处理长期依赖关系。

### 数学原理

**1. 遗忘门（Forget Gate）**

遗忘门控制着当前单元状态中哪些信息需要被丢弃。遗忘门的输出是一个 0 到 1 之间的值，表示每个信息保留的程度。

$$
f^{(t)} = \sigma\bigl(W_f \,[h^{(t-1)},\, x^{(t)}] + b_f\bigr)
$$

* $$f^{(t)}$$ 是遗忘门的输出。
* $$\sigma$$ 是 sigmoid 激活函数。
* $$W_f$$ 是遗忘门的权重矩阵。
* $$[h^{(t-1)},\, x^{(t)}]$$ 表示上一个时刻的隐藏状态和当前输入的拼接向量。
* $$b_f$$ 是遗忘门的偏置项。

***

**2. 输入门（Input Gate）**

输入门控制着新信息的加入程度。它包括两个部分：一个 sigmoid 层决定需要更新的信息，另一个 tanh 层生成新的候选记忆单元。

$$
i^{(t)} = \sigma\bigl(W_i \,[h^{(t-1)},\, x^{(t)}] + b_i\bigr)
\quad
\tilde{c}^{(t)} = \tanh\bigl(W_c \,[h^{(t-1)},\, x^{(t)}] + b_c\bigr)
$$

* $$i^{(t)}$$ 是输入门的输出。
* $$\tilde{c}^{(t)}$$ 是新的候选记忆单元。
* $$W_i, W_c$$ 分别是输入门和候选记忆单元的权重矩阵。
* $$b_i, b_c$$ 分别是输入门和候选记忆单元的偏置项。

***

**3. 更新记忆单元状态**

通过遗忘门和输入门的作用，更新当前的记忆单元状态：

$$
c^{(t)} = f^{(t)} \odot c^{(t-1)} + i^{(t)} \odot \tilde{c}^{(t)}
$$

* $$c^{(t)}$$ 是当前时刻的记忆单元状态。
* $$c^{(t-1)}$$ 是上一个时刻的记忆单元状态。
* $$\odot$$ 表示逐元素相乘。

***

**4. 输出门（Output Gate）**

输出门控制着当前时刻的隐藏状态输出。它决定哪些部分的记忆单元状态将被输出。

$$
o^{(t)} = \sigma\bigl(W_o \,[h^{(t-1)},\, x^{(t)}] + b_o\bigr)
\quad
h^{(t)} = o^{(t)} \odot \tanh\bigl(c^{(t)}\bigr)
$$

* $$o^{(t)}$$ 是输出门的输出。
* $$h^{(t)}\$$ 是当前时刻的隐藏状态。
* $$W_o$$ 是输出门的权重矩阵。
* $$b_o$$ 是输出门的偏置项。

***

**LSTM 的算法流程**

下面是 LSTM 在处理时间序列数据时的详细算法流程：

1. **初始化：**\
   初始化所有权重矩阵 $$\{W_f, W_i, W_c, W_o\}$$ 和偏置项 $$\{b_f, b_i, b_c, b_o\}$$。
2. **输入数据：**\
   输入时间序列数据 $$\{x^{(1)}, x^{(2)}, \dots, x^{(T)}\}$$。
3.  **计算遗忘门输出：**

    $$
    f^{(t)} = \sigma\bigl(W_f [h^{(t-1)}, x^{(t)}] + b_f\bigr)
    $$
4.  **计算输入门输出：**

    $$
    i^{(t)} = \sigma\bigl(W_i [h^{(t-1)}, x^{(t)}] + b_i\bigr)
    $$
5.  **生成候选记忆单元：**

    $$
    \tilde{c}^{(t)} = \tanh\bigl(W_c [h^{(t-1)}, x^{(t)}] + b_c\bigr)
    $$
6.  **更新记忆单元状态：**

    $$
    c^{(t)} = f^{(t)} \odot c^{(t-1)} + i^{(t)} \odot \tilde{c}^{(t)}
    $$
7.  **计算输出门输出：**

    $$
    o^{(t)} = \sigma\bigl(W_o [h^{(t-1)}, x^{(t)}] + b_o\bigr)
    $$
8.  **计算当前隐藏状态：**

    $$
    h^{(t)} = o^{(t)} \odot \tanh\bigl(c^{(t)}\bigr)
    $$
9. **输出隐藏状态** $$h^{(t)}$$ **并更新内部状态** $$\{h^{(t)}, c^{(t)}\}$$**。**

LSTM 通过遗忘门、输入门、候选记忆单元和输出门来控制信息的流动和更新，克服了传统 RNN 中梯度消失和梯度爆炸的问题，使其能够更好地捕捉长时间依赖关系。其核心思想是通过门控机制选择性地保留和更新信息，从而实现对时间序列数据的有效建模和预测。

## CNN

Convolutional Neural Network，卷积神经网络，专门用于处理图像和视频数据。它通过卷积层（Convolutional Layers）自动提取特征，避免了传统机器学习中手动设计特征的过程。主要由 卷积层（Convolutional Layer）、池化层（Pooling Layer）和全连接层（Fully Connected Layer） 组成。

### 数学原理

#### CNN 的输入表示

输入通常是三维张量：

$$
\text{Input shape: } (H, W, C)
$$

* $$H$$：图像高度
* $$W$$：图像宽度
* $$C$$：通道数（如 RGB 图像为 3）

***

#### 卷积层

**卷积运算：**

$$
y_{i,j} = \sum_m \sum_n x_{i+m,j+n} \cdot w_{m,n} + b
$$

其中：

* $$x$$：输入图像，$$w$$：卷积核，$$b$$：偏置项
* $$y_{i,j}$$：输出特征图上的第 $$i,j$$ 个元素

***

本质上是用一个小的「滤波器」在图像上滑动，并计算局部区域的加权和

**卷积核（Filter）：**

* 通常为 $$3 \times 3$$、$$5 \times 5$$大小
* 输入通道为 $$C$$，则卷积核尺寸为 $$C \times K \times K$$
* 参数通过反向传播自动学习

***

**步长（Stride）与填充（Padding）：**

* **步长 Stride**：卷积核每次移动的步数（如 1, 2）
* **填充 Padding**：为了防止卷积后图像尺寸缩小，可以在输入图像边缘填充p个像素
  * "valid"：不填充，输出尺寸变小
  * "same"：填充使输出尺寸等于输入尺寸 $$P = \frac{K-1}{2}$$

***

#### 激活函数（Activation Function）

CNN 中最常用的是 ReLU（Rectified Linear Unit）：

$$
\text{ReLU}(x) = \max(0, x)
$$

作用：

* 增加非线性
* 加速收敛
* 缓解梯度消失问题

***

#### 池化层（Pooling Layer）

池化层用于降低特征图的维度，同时保留重要信息。

**最大池化（Max Pooling）**

$$
O(i,j)=\max_{(m,n)\in P}I(i+m,j+n)
$$

**平均池化（Average Pooling）**

$$
O(i,j)=\frac{1}{|P|}\sum_{(m,n)\in P}I(i+m,j+n)
$$

#### 扁平化（Flatten）

将卷积/池化后的输出展平为一维向量：

$$
\text{Tensor shape } (C, H, W) \Rightarrow \text{Vector shape } (C \times H \times W)
$$

***

#### Softmax 分类层

将最后的全连接输出变为概率分布：

$$
\text{Softmax}(z_i) = \frac{e^{z_i}}{\sum_{j} e^{z_j}}
$$

其中：

* $$z$$：FC 层输出向量
* 输出为每个类别的概率

***

### CNN 的训练流程

1. **前向传播**：依次计算各层输出，得到预测值$$L(y,\hat{y})$$
2. **计算损失**：通常使用交叉熵损失函数常用 **交叉熵损失（Cross Entropy Loss）**$$\mathcal{L} = - \sum_{i=1}^C y_i \log(\hat{y}_i)$$

* $$y$$：真实标签的 one-hot 编码
* $$\hat{y}$$：预测的概率分布

3. **反向传播**：计算梯度(使用链式法则)，传播误差
4. **参数更新**：使用梯度下降（如 SGD, Adam）更新参数

***

### 总结

* 卷积层（Convolutional Layer）- 提取局部特征
  * 模拟人类视觉的“看局部”能力，使用滤波器（Kernel）滑动图像区域，提取边缘、纹理、颜色等信息。
  * 每次滑动执行局部加权求和（卷积操作），提取不同层次特征。
* 全连接层（Fully Connected Layer, FC）- 做最终分类决策
  * 将前面提取的特征展平为一维向量
  * 输入到传统神经网络层中，输出分类结果
* &#x20;池化层（Pooling Layer）- 缩小数据，保留关键信息
  * 通过 **最大池化（Max Pooling）** 或 **平均池化（Average Pooling）** 缩小特征图。类似于「压缩图片，但保留主要内容」
  * 保留最重要的特征，并减少计算复杂度
* Softmax 层：输出为各类概率分布
* 通过反向传播 + 优化器训练整个网络



## Transform

由两个主要部分组成：编码器（Encoder）和解码器（Decoder），编码器就像一个阅读器，它把输入的信息（比如一句话）进行理解和抽象；而解码器则像一个说话器，把这种抽象的理解重新生成成另一种形式（比如另一种语言的翻译）

编码器和解码器都不是简单的单层结构，而是堆叠了多层，每一层都做一些类似的信息加工。

每一层内部的主要工作就是通&#x8FC7;**“注意力机制”（Attention）**&#x6765;确定哪些信息是关键，应该被重点关注，同时还会通过一些数学运算对信息进行转化和增强。具体来说：

* **输入表示**：把文本数据先变成数字化的向量表示，并加入位置信息（因为模型本身不具备位置信息的顺序概念）。
* **编码器堆叠**：多层编码器各自关注不同层次的信息特征，通过注意力机制互相“对话”，提取全局和局部的重要信息。
* **解码器堆叠**：解码器同样有多层，它不仅关注已经生成的内容，还从编码器那里获取输入的关键信息，通过注意力机制将这两部分信息融合，生成最终的输出。
* **输出生成**：经过解码器处理，最终生成所需的目标序列，比如翻译后的句子或其他任务的输出。



