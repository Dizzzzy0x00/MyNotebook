# Deep Learning

## RNN

Recurrent Neural Network，循环神经网络，用于处理**序列数据**的神经网络。RNN具有内部记忆功能，可以在序列中的当前步骤依赖于前一步骤的信息。这使得RNN特别适合于处理诸如时间序列、语言文本、语音识别等需要考虑上下文的任务。

### RNN 的数学原理和算法流程

#### 1. 基本结构

RNN 的基本单元由一个循环结构组成，它允许信息在网络的时间步骤之间传递。假设我们有一个输入序列  $$({x^{(1)}, x^{(2)}, \dots, x^{(T)}})$$，RNN 通过以下公式来更新其隐藏状态和输出：

* **隐藏状态更新：**\

  \
  $$h^{(t)} = f\bigl(W_{hh} h^{(t-1)} + W_{xh} x^{(t)} + b_h\bigr)$$
  \

  \
  其中：
  * $$h^{(t)}$$是时间步 (t) 的隐藏状态。
  * $$W_{hh}$$是隐藏状态到隐藏状态的权重矩阵，维度为 $$\mathbb{R}^{H\times H}$$。
  * $$W_{xh}$$是输入到隐藏状态的权重矩阵，维度为 $$\mathbb{R}^{D\times H}$$。
  * $$b_h$$ 是隐藏层的偏置项，维度为  $$\mathbb{R}^{H}$$。
  * $$f(x)$$是激活函数通常是  $$\tanh$$或 $$ReLU$$）。
* **输出计算：**\
  $$y^{(t)} = g\bigl(W_{hy} h^{(t)} + b_y\bigr)$$
  \
  其中：
  * &#x20;$$y^{(t)}$$ 是时间步 t 的输出。
  * &#x20;$$W_{hy}$$ 是隐藏状态到输出的权重矩阵，维度为 $$\mathbb{R}^{H\times O}$$。
  * $$b_y$$是输出层的偏置项，维度为 $$\mathbb{R}^{O}$$。
  * $$g$$是输出的激活函数（根据任务不同，可能是 softmax、sigmoid 等）。

***

####

#### 2. 前向传播过程

1. **初始化隐藏状态：**\
   $$h^{(0)} = \mathbf{0} \quad\text{或随机初始化}$$\
2. **通过时间步更新隐藏状态和输出：**\
   对于每个时间步 $$t=1,2,\dots,T$$：\
   * 计算隐藏状态：\
     $$h^{(t)} = f\bigl(W_{hh} h^{(t-1)} + W_{xh} x^{(t)} + b_h\bigr)$$\
   * 计算输出：\
     $$y^{(t)} = g\bigl(W_{hy} h^{(t)} + b_y\bigr)$$  

***

####

#### 3. 损失函数和反向传播过程

* **总损失：**\
  $$\mathcal{L} = \sum_{t=1}^{T} \ell\bigl(y^{(t)},\, \hat{y}^{(t)}\bigr)$$\
  其中：\
  * $$\ell$$ 是每个时间步的损失函数（如交叉熵损失）。\
  * $$\hat{y}^{(t)}$$ 是时间步 \(t\) 的真实值。  
* **BPTT（Backpropagation Through Time）：**  
  1. **初始化梯度：**\
     $$\frac{\partial \mathcal{L}}{\partial W_{xh}} = 0,\quad
       \frac{\partial \mathcal{L}}{\partial W_{hh}} = 0,\quad
       \frac{\partial \mathcal{L}}{\partial W_{hy}} = 0,\quad
       \frac{\partial \mathcal{L}}{\partial b_h} = 0,\quad
       \frac{\partial \mathcal{L}}{\partial b_y} = 0.$$\
  2. **反向传播误差（从 \(t=T\) 到 \(t=1\)）：**\
     * 输出层梯度：\
       $$\delta_y^{(t)} = \frac{\partial \ell\bigl(y^{(t)}, \hat{y}^{(t)}\bigr)}{\partial z^{(t)}},\quad
         z^{(t)} = W_{hy} h^{(t)} + b_y.$$\
     * 隐藏层梯度：\
       $$\delta_h^{(t)} = \bigl(W_{hy}^\top \delta_y^{(t)} + W_{hh}^\top \delta_h^{(t+1)}\bigr) \odot f'\bigl(a^{(t)}\bigr),$$\
       其中 $$a^{(t)} = W_{hh} h^{(t-1)} + W_{xh} x^{(t)} + b_h$$。\
     * 累计权重梯度：\
       $$
       \begin{aligned}
         \frac{\partial \mathcal{L}}{\partial W_{hy}} &\mathrel{+}= \delta_y^{(t)}\,h^{(t)\top},\\
         \frac{\partial \mathcal{L}}{\partial b_y}  &\mathrel{+}= \delta_y^{(t)},\\
         \frac{\partial \mathcal{L}}{\partial W_{hh}} &\mathrel{+}= \delta_h^{(t)}\,h^{(t-1)\top},\\
         \frac{\partial \mathcal{L}}{\partial W_{xh}} &\mathrel{+}= \delta_h^{(t)}\,x^{(t)\top},\\
         \frac{\partial \mathcal{L}}{\partial b_h}  &\mathrel{+}= \delta_h^{(t)}.
       \end{aligned}
       $$\
  3. **参数更新（梯度下降）：**\
     $$
     \begin{aligned}
       W_{xh} &\leftarrow W_{xh} - \eta \frac{\partial \mathcal{L}}{\partial W_{xh}},\\
       W_{hh} &\leftarrow W_{hh} - \eta \frac{\partial \mathcal{L}}{\partial W_{hh}},\\
       W_{hy} &\leftarrow W_{hy} - \eta \frac{\partial \mathcal{L}}{\partial W_{hy}},\\
       b_h    &\leftarrow b_h    - \eta \frac{\partial \mathcal{L}}{\partial b_h},\\
       b_y    &\leftarrow b_y    - \eta \frac{\partial \mathcal{L}}{\partial b_y}.
     \end{aligned}
     $$
####

RNN 通过循环结构捕捉序列中的依赖关系，通过前向传播计算隐藏状态和输出，并通过时间反向传播来更新权重，从而在处理序列数据时有效利用上下文信息。  
