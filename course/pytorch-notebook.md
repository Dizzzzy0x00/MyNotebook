# Pytorch Notebook

### Neural Network&#x20;

#### 训练，验证，测试集（Train / Dev / Test sets）

在机器学习中，通常将样本分成三部分，一部分作为训练集，一部分作为简单交叉验证集，有时也称之为验证集，方便起见，我就叫它验证集（**dev set**），其实都是同一个概念，最后一部分则作为测试集——即**训练集，验证集和测试集**

数据集规模相对较小，适用传统的划分比例（常见做法是将所有数据三七分，就是70%验证集，30%测试集，如果没有明确设置验证集，也可以按照60%训练，20%验证和20%测试集来划分），数据集规模较大的，验证集和测试集要小于数据总量的20%或10%。



#### 偏差，方差（Bias /Variance）

以下图数据集为例





如果给数据集拟合一条直线，可能得到一个逻辑回归拟合，但它并不能很好地拟合该数据，这是高偏差（**high bias**）的情况，我们称为“欠拟合”（**underfitting**）。

相反的如果我们拟合一个非常复杂的分类器，比如深度神经网络或含有隐藏单元的神经网络，可能就非常适用于这个数据集，但是这看起来也不是一种很好的拟合方式分类器方差较高（**high variance**），数据过度拟合（**overfitting**）。

在两者之间，可能还有一些像图中这样的，复杂程度适中，数据拟合适度的分类器，这个数据拟合看起来更加合理，我们称之为“适度拟合”（**just right**）是介于过度拟合和欠拟合中间的一类。



#### 正则化（Regularization）

{% embed url="https://blog.csdn.net/m0_64524798/article/details/129330847" %}

对于high variance带来的过拟合问题，一个简单可靠的解决方式是正则化

通过向损失函数添加额外的项（惩罚项）来实现，这样可以在最小化目标函数的同时，控制模型的复杂度

**L1范数和L2范数**

* 机器学习中几乎都可以看到损失函数后面会添加一个额外项，常用的额外项一般有两种，称作 L1正则化 和 L2正则化，或者 L1范数 和 L2范数，是为了限制模型的参数，防止模型过拟合而加在损失函数后面的一项。
* L1是模型各个参数的绝对值之和，L2是模型各个参数的平方和的开方值
* L1： 为模型加入先验， 简化模型， 使权值稀疏，由于权值的稀疏，从而过滤掉一些无用特征，防止过拟合
* L2： 根据L2的特性，它会使得权值减小，即使平滑权值，一定程度上也能和L1一样起到简化模型，加速训练的作用，同时可防止模型过拟合

如果用的是`L1`正则化，最终`W`会是稀疏的，也就是说`w`向量中有很多0，有人说这样有利于压缩模型，因为集合中参数均为0，存储模型所占用的内存更少。实际上，虽然`L1`正则化使模型变得稀疏，却没有降低太多存储内存，人们在训练网络时，越来越倾向于使用`L2`正则化

**Dropout正则化**

* 缓解过拟合问题
* 输入的特征都存在被随机清除的可能，所以该神经元不会再特别依赖于任何一个输入特征，也就是不会给任何一个输入特征设置太大的权重

#### Batch Normalization （批量归一化）

* 控制数据的分布，加快网络的收敛



\


**神经元**：最小的神经网络，将若干个输入加权重再加上偏置得到输出

W是权重`(W1,W2,W3)`，x是特征值`(x1,x2,x3)`，f是激活函数（一般是非线性），b是偏置

$$
h_{W,b}(x) = f(W ^ {T}x)=f(\sum_{i=1}^{3} W_{i}x_{i}+b)
$$

**激活函数**：sigmoid，二分类逻辑斯蒂回归模型

$$
f(x) = \frac{1}{1+e^{-x}}
$$

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

激活函数的选择：

对于隐藏层:

1. 优先选择RELU激活函数
2. 如果ReLu效果不好，那么尝试其他激活，如Leaky ReLu等。
3. 如果你使用了Relu， 需要注意一下Dead Relu问题， 避免出现大的梯度从而导致过多的神经元死亡。
4. 不要使用sigmoid激活函数，可以尝试使用tanh激活函数

对于输出层

1. 二分类问题选择sigmoid激活函数
2. 多分类问题选择softmax激活函数
3. 回归问题选择identity激活函数

多神经元——>多分类任务

* W从向量扩展为矩阵
* 输出W\*x变成向量
* 多输出神经元——>softmax——>多分类逻辑斯蒂回归模型
*

    <figure><img src="../.gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

#### 梯度下降算法的优化方法

**指数加权平均**

* 指数移动加权平均则是参考各数值，并且各数值的权重都不同，**距离越远的数字对平均数计算的贡献就越小**（权重较小），距离越近则对平均数的计算贡献就越大（权重越大）

**Momentum**

* 当梯度下降碰到 “峡谷” 、”平缓”、”鞍点” 区域时, 参数更新速度变慢. Momentum 通过指数加权平均法，**累计历史梯度值**，进行参数更新，越近的梯度值对当前参数更新的重要性越大
* 梯度下降公式中梯度的计算，就不再是当前时刻 t 的梯度值，而是**历史梯度值的指数移动加权平均值**
* 当处于鞍点位置时，由于当前的梯度为 0，参数无法更新。但是 Momentum 动量梯度下降算法已经在先前积累了一些梯度值，很有可能使得跨过鞍点
* Momentum 使用移动加权平均，平滑了梯度的变化，使得前进方向更加平缓，有利于加快训练过程。一定程度上有利于降低 “峡谷” 问题的影响，但Momentum 并没有学习率进行优化

**AdaGrad**

* **对不同的参数分量使用不同的学习率**，AdaGrad 的学习率总体会逐渐减小，这是因为 AdaGrad 认为：在起初时，我们距离最优目标仍较远，可以使用较大的学习率，加快训练速度，随着迭代次数的增加，学习率逐渐下降
* 缺点是可能会使得学习率过早、过量的降低，导致模型训练后期学习率太小，较难找到最优解

**RMSProp**

* 对 AdaGrad 的优化. 最主要的不同是，其**使用指数移动加权平均梯度替换历史梯度的平方和**
* 通过引入**衰减系数 β**，控制历史梯度对历史梯度信息获取的多少，被证明在神经网络非凸条件下的优化更好，学习率衰减更加合理一些

**Adam**

* Momentum 使用指数加权平均计算当前的梯度值、AdaGrad、RMSProp 使用自适应的学习率，Adam 结合了 Momentum、RMSProp 的优点，使用：移动加权平均的梯度和移动加权平均的学习率。使得能够自适应学习率的同时，也能够使用 Momentum 的优点

#### 参数初始化 <a href="#id-5" id="id-5"></a>

* 均匀分布初始化,权重参数初始化从区间均匀随机取值。即在`(-1/√d,1/√d)`均匀分布中生成当前神经元的权重，其中`d`为每个神经元的输入数量
* 正态分布初始化, 随机初始化从均值为0，标准差是1的高斯分布中取样，使用一些很小的值对参数W进行初始化
* 全0初始化，将神经网络中的所有权重参数初始化为 0
* 全1初始化，将神经网络中的所有权重参数初始化为 1
* 固定值初始化，将神经网络中的所有权重参数初始化为某个固定值
* kaiming 初始化，也叫做 HE 初始化. HE 初始化分为正态分布的 HE 初始化、均匀分布的 HE 初始化.
* xavier 初始化，也叫做Glorot初始化，该方法的基本思想是各层的激活值和梯度的方差在传播过程中保持一致。它有两种，一种是正态分布的 xavier 初始化、一种是均匀分布的 xavier 初始化

#### [正则化](https://blog.csdn.net/m0\_64524798/article/details/129330847)



#### 卷积神经网络进阶

AlexNet

* 第一个卷积层，输入224\*224，步长Stride =4，卷积核11\*11，padding=3——>输出大小：216/4+1=5
* 首个使用Relu激活函数的网络，训练速度很快
* 2个GPU并行
* 1，2，5卷积层后跟随Max-Pooling层
* 全连接层：随机dropout
  * 为什么使用dropout在全连接层：全连接层含有大量参数，容易过拟合
  * 每次dropouut相当于训练了一个子网络，最后的结果相当于很多子网络组合
  * dropout = 0.5
* 数据增强：图片随机采样：256\*256采样224\*224
* Batch size =128，SGD momentum = 0.9，Learning rate = 0.01，过一定次数之后降低为1/10
* 7个CNN组合投票

VGGNet

* 更大更深的网络
* 训练技巧：先训练浅层网络，再去训练深层网络，对于不同尺度的输入训练不同的分类器

InceptionNet

* 解决深层网络容易过拟合和更大计算量的的问题（稀疏网络虽然减少了参数但并没有减少计算量）
* 分组卷积

ResNet

* 之前的网络模型深度达到某个程度以后继续训练会导致训练集准确率下降
*   $$F(x) + identity(x)$$，F(x)是残差学习，identity是恒等变换，保证深层网络至少能和浅层效果持平，残差结构使得每一层的数据分布接近，容易学习



MobileNet

* 引入深度可分离卷积

NASNet 自学习



### Pytorch基本语法

创建矩阵

<pre class="language-python"><code class="lang-python"><strong>from __future__ import print_function
</strong><strong>import torch
</strong>x = torch.empty(5,3) #没有初始化的矩阵 
y = torch.rand(5,3) #随机初始化的矩阵
#对比有无初始化的矩阵，未初始化的矩阵本身不包含任何确切的值，是分配时内存的脏数据
<strong>print(y)
</strong><strong>
</strong><strong>
</strong><strong>#输出：
</strong>tensor([[0.0802, 0.2668, 0.9357],
        [0.0862, 0.7464, 0.1324],
        [0.9528, 0.8597, 0.0585],
        [0.5185, 0.9477, 0.8906],
        [0.3010, 0.7061, 0.6548]])
        
#创建全零的矩阵并指定数据元素的类型
x = torch.zeros(5,3,dtype = torch.long) #使用torch.来指定类型dtype

#创建全1的矩阵并设置requires_grad
x = torch.ones(2, 2, requires_grad=True)
print(x)
#输出：
tensor([[1., 1.],
        [1., 1.]], requires_grad=True)
        
y = x + 2
print(y)
#输出：
tensor([[3., 3.],
        [3., 3.]], grad_fn=&#x3C;AddBackward0>)
        
</code></pre>

基本运算

```python
torch.add(x,y)
torch.add(x,y,out=result)
y.add_(x)
```

```python
x.view() #改变张量尺寸

Torch Tensor和Numpy Array转换：
b = a.numpy() #b和a共享底层内存
b = torch.from_numpy(a) #b和a共享底层内存
```

autograd

如果设置了requires\_grad=True，将追踪在这个类上定义的所有操作，当代码进行反向传播时，直接调用.backward()就可以自动计算所有的梯度，在这个Tensor上所有的梯度将累加到属性grad中

.detach()将tensor从计算图中删除，未来的回溯计算中也不会计算

<pre class="language-python"><code class="lang-python"><strong>y = x + 2
</strong><strong>z = y * y * 3
</strong><strong>out = z.mean()
</strong><strong>print(x.grad_fn)
</strong>print(y.grad_fn)
print(z.grad_fn)
print(out.grad_fn)

#输出：
None
&#x3C;AddBackward0 object at 0x000001F7AC4A7AF0>
&#x3C;MulBackward0 object at 0x000001F7AC4A7760>
&#x3C;MeanBackward0 object at 0x000001F7AC4A7A90>
</code></pre>

### 一个最简单的神经网络

PyTorch中已经为我们准备好了现成的网络模型，只要继承nn.Module，并实现它的forward方法，PyTorch会根据autograd，自动实现backward函数，在forward函数中可使用任何tensor支持的函数，还可以使用if、for循环、print、log等Python语法，写法和标准的Python写法一致。

<pre class="language-python"><code class="lang-python"><strong>
</strong><strong># 首先要引入相关的包
</strong>import torch
# 引入torch.nn并指定别名
import torch.nn as nn

import torch.nn.functional as F

device = torch.device("cuda" if torch.cuda.is_available() else "cpu")#优先使用GPU
</code></pre>

<pre class="language-python"><code class="lang-python">class Net(nn.Module):
    def __init__(self):
        # nn.Module子类的函数必须在构造函数中执行父类的构造函数
        super(Net, self).__init__()
<strong>
</strong>        # 卷积层 '1'表示输入图片为单通道， '6'表示输出通道数，'3'表示卷积核为3*3
        self.conv1 = nn.Conv2d(1, 6, 3) 
        #线性层，输入1350个特征，输出10个特征
        self.fc1   = nn.Linear(1350, 10) 
         #这里的1350是如何计算的呢？这就要看后面的forward函数
    #正向传播 
    def forward(self, x): 
        print(x.size()) # 结果：[1, 1, 32, 32]
        # 卷积 -> 激活 -> 池化 
        x = self.conv1(x) #根据卷积的尺寸计算公式，计算结果是30
        x = F.relu(x)
        print(x.size()) # 结果：[1, 6, 30, 30]
        x = F.max_pool2d(x, (2, 2)) #我们使用池化层，计算结果是15
        x = F.relu(x)
        print(x.size()) # 结果：[1, 6, 15, 15]
        # reshape，‘-1’表示自适应
        #这里做的就是压扁的操作 就是把后面的[1, 6, 15, 15]压扁，变为 [1, 1350]
        x = x.view(x.size()[0], -1) 
        print(x.size()) # 这里就是fc1层的的输入1350 
        x = self.fc1(x)        
        return x

net = Net()
print(net)
</code></pre>

```python
import torch.nn as nn
#torch.nn只支持mini-batches，不支持一次只输入一个样本，即一次必须是一个batch

import torch
import torch.nn.functional as F

class Net(nn.Module):
    def __init__(self):
        super(Net,self).__init__()
        self.conv1 = nn.Conv2d(1,6,3)
        self.conv2 = nn.Conv2d(6,16,3)
        self.fc1   = nn.Linear(16 * 6 * 6,120) 
        #输入图片是32*32的，经过nn.Conv2d(1,6,3)后变为16*6*6
        self.fc2   = nn.Linear(120,84)
        self.fc3   = nn.Linear(84,10) #最终10是十分类
    def forward(self,x):
        x = F.max_pool2d(F.relu(self.conv1(x)),(2,2))
        x = F.max_pool2d(F.relu(self.conv2(x)),2)
        x = x.view(-1,self.num_flat_features(x))
        x = F.relu(self.fc1(x))
        x = F.relu(self.fc2(x))
        x = self.fc3(x)
        return x
    def num_flat_features(self,x):
        size = x.size()[1:]
        num_features = 1
        for s in size:
            num_features *= s
        return num_features

net = Net()
print(net)

params = list(net.parameters()) #所有可训练的参数
print(len(params))
print(params[0].size())

input = torch.randn(1,1,32,32)
out = net(input)
print(out)
print(out.size())
```

<pre><code>输出：
Net(
  (conv1): Conv2d(1, 6, kernel_size=(3, 3), stride=(1, 1))
  (conv2): Conv2d(6, 16, kernel_size=(3, 3), stride=(1, 1))
  (fc1): Linear(in_features=576, out_features=120, bias=True)
  (fc2): Linear(in_features=120, out_features=84, bias=True)
<strong>  (fc3): Linear(in_features=84, out_features=10, bias=True)
</strong>)
10
torch.Size([6, 1, 3, 3])
tensor([[-0.0268,  0.0764, -0.0940,  0.1138, -0.0782,  0.1017, -0.0789,  0.0946,
         -0.0083,  0.0594]], grad_fn=&#x3C;AddmmBackward0>)
torch.Size([1, 10])
</code></pre>

#### 参数初始化

```python
import torch
import torch.nn.functional as F
import torch.nn as nn


# 1. 均匀分布随机初始化
def test01():

    linear = nn.Linear(5, 3)
    # 从0-1均匀分布产生参数
    nn.init.uniform_(linear.weight)
    print(linear.weight.data)


# 2. 固定初始化
def test02():

    linear = nn.Linear(5, 3)
    nn.init.constant_(linear.weight, 5)
    print(linear.weight.data)


# 3. 全0初始化
def test03():

    linear = nn.Linear(5, 3)
    nn.init.zeros_(linear.weight)
    print(linear.weight.data)


# 4. 全1初始化
def test04():

    linear = nn.Linear(5, 3)
    nn.init.ones_(linear.weight)
    print(linear.weight.data)


# 5. 正态分布随机初始化
def test05():

    linear = nn.Linear(5, 3)
    nn.init.normal_(linear.weight, mean=0, std=1)
    print(linear.weight.data)


# 6. kaiming 初始化
def test06():

    # kaiming 正态分布初始化
    linear = nn.Linear(5, 3)
    nn.init.kaiming_normal_(linear.weight)
    print(linear.weight.data)

    # kaiming 均匀分布初始化
    linear = nn.Linear(5, 3)
    nn.init.kaiming_uniform_(linear.weight)
    print(linear.weight.data)


# 7. xavier 初始化
def test07():

    # xavier 正态分布初始化
    linear = nn.Linear(5, 3)
    nn.init.xavier_normal_(linear.weight)
    print(linear.weight.data)

    # xavier 均匀分布初始化
    linear = nn.Linear(5, 3)
    nn.init.xavier_uniform_(linear.weight)
    print(linear.weight.data)
```

```python

target = torch.randn(10)
target = target.view(1,-1)
criterion = nn.MSELoss()

loss = criterion(out,target)
#首先进行梯度清零
net.zero_grad()
print('bias.grad before backward:',net.conv1.bias.grad)
loss.backward()
print('bias.grad after  backward:',net.conv1.bias.grad)

#输出：
#bias.grad before backward: None
#bias.grad after  backward: tensor([ 9.3509e-03, -2.6641e-02, -7.2068e-03, -5.2125e-05,  1.4694e-02,
#        -1.1412e-03])
```

#### 更新网络参数

```python
import torch.optim as optim
#SGD随机梯度下降，用optima创建优化器对象
optimizer = optim.SGD(net.parameters(),lr = 0.01)#学习率和优化参数
optimizer.zero_grad()
output = net(input)
loss = criterion(output,target)
loss.backward() #反向传播
optimizer.step() #参数的更新
```

## NLP

### 文本预处理

#### jieba分词

精确模式分词(默认)

```python
import jieba
content = "测试文本样例，用于jieba分词测试"
jieba.cut(content, cua_all=False) #返回一个生成器对象，cua_all=False表示以精确模式进行切割
#输出：<generator object Tokenizer.cut at 0x0000022E57897DD0>
jieba.lcut(content,cut_all=False) #返回一个列表对象，cua_all=False表示以精确模式进行切割
#输出：['测试', '文本', '样例', '，', '用于', 'jieba', '分词', '测试']
```

全模式分词：把句子中所有能进行分词的都进行分词“

```python
jieba.lcut(content,cut_all=False)
#['测试', '文本', '样', '例', '，', '用于', 'jieba', '分词', '测试']
```

搜索引擎分词

```python
jieba.lcut_for_search(content)
```

其他

```
jieba.load_userdict("./userdict.txt") #导入用户自定义的词典
#词典格式: 每一行分三部分：词语、词频（可省略）、词性（可省略），用空格隔开，顺序不可颠倒。
#词典样式：
云计算 5 n
李小福 2 nr
easy_install 3 eng
好用 300
韩玉赏鉴 3 nz
八一双鹿 3 nz
```

#### 中英文分词工具hanlp

基于tensorflow2.0

```python
import hanlp
tokenizer = hanlp.load('CTB6_CONVSEG') #导入预训练模型
```

#### 命名实体识别（NER）

命名实体：人名 / 地名 / 机构等专有名词统称为命名实体

```python
#使用hanlp进行中文命名实体识别
reconginzer  = hanlp.laod(hanlp.pretrained.ner.MSRA_NER_BERT_BASE_ZH)
#注意这个模型的输入是列表：
print(recongnizer(list('成都帕鲁工业公司董事长阿杜和秘书阿斯发来到美国纽约现代艺术博物馆参观')))
#使用hanlp进行英文命名实体识别：
reconginzer  = hanlp.laod(hanlp.pretrained.ner.MSRA_NER_BERT_BASE_EN)
```



#### 文本张量表示方法

**one-hot词向量表示**

又称独热编码，将每个词表示成具有n个元素的向量，这个词向量中只有一个元素是1，其他元素都是0，不同词汇元素为0的位置不同，其中n的大小是整个语料中不同词汇的总数

```python
# 导入用于对象保存与加载的joblib
import joblib
# 导入keras中的词汇映射器Tokenizer
from keras.preprocessing.text import Tokenizer
# 假定vocab为语料集所有不同词汇集合
vocab = {"周杰伦", "陈奕迅", "王力宏", "李宗盛", "吴亦凡", "鹿晗"}
# 实例化一个词汇映射器对象
t = Tokenizer(num_words=None, char_level=False)
# 使用映射器拟合现有文本数据
t.fit_on_texts(vocab)

for token in vocab:
    zero_list = [0]*len(vocab)
    # 使用映射器转化现有文本数据, 每个词汇对应从1开始的自然数
    # 返回样式如: [[2]], 取出其中的数字需要使用[0][0]
    token_index = t.texts_to_sequences([token])[0][0] - 1
    zero_list[token_index] = 1
    print(token, "的one-hot编码为:", zero_list)

# 使用joblib工具保存映射器, 以便之后使用
tokenizer_path = "./Tokenizer"
joblib.dump(t, tokenizer_path)

```

**word2vec**

将词汇表示成向量的无监督训练方法, 该过程将构建神经网络模型, 将网络参数作为词汇的向量表示, 它包含CBOW和skipgram两种训练模式

* CBOW(Continuous bag of words)模式
  * 给定一段用于训练的文本语料, 再选定某段长度(窗口)作为研究对象, **使用上下文词汇预测目标词汇**
  *

      <figure><img src="../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>
* skipgram模式
  * 给定一段用于训练的文本语料, 再选定某段长度(窗口)作为研究对象, **使用目标词汇预测上下文词汇**
  *

      <figure><img src="../.gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>

#### 使用fasttext工具实现word2vec的训练和使用

### 经典的序列模型

#### HMM

HMM是隐含马尔可夫模型，一般以文本序列数据为输入，以该序列对应的隐含序列为输出（序列数据中的每个单元的隐性信息），肉眼可见的**文本内容就是观测序列**，如果将这句话以词为单位进行划分，得到一个词序列，那么每个词对应的词性构成的**词性序列就是一个隐含序列**

HMM在NLP领域可以解决文本序列标注问题（分词/词性标注/命名实体识别等）

* HMM模型表示：`lambda = HMM(A,B,pi)`,A是转移概率矩阵，B是发射概率矩阵，pi是初始概率矩阵
* 准备观测序列和对应的隐含序列，通过不断训练，使由观测序列到对应的隐含序列的概率最大
* 马尔可夫假设：隐含序列中的每个单元的可能性只与上一个单元有关
* 使用维特比算法从隐含序列的条件概率中找出概率最大的一条序列路径就是我们需要的隐含序列
* _维特比算法_(Viterbialgorithm)是一种经典的动态规划算法,用于寻找最有可能产生观测事件_序列的_维特比路径,即隐含状态序列

#### CRF

CRF：条件随机场

* CRF模型表示：`lambda = CRF(W1,W2,W3,...,Wn),Wi是模型参数`
* 训练CRF模型语料同样是观测序列及其对应的隐含序列，但还需要做人工特征工程，通过不断训练求得一组参数，使由观测序列到对应隐含序列的概率最大
* 最后，还是使用维特比算法从隐含序列的条件概率分布中找出概率最大的一条序列路径就是需要的隐含序列

**HMM模型存在隐马假设，**而CRF不存在，因此**HMM的计算速度要比CRF模型快很多**，适用于对预测性能要求较高的场合



### RNN

RNN：循环神经网络，RNN的循环机制使模型隐层**上一时间步产生的结果能够作为当下时间步输入的一部分**(即当下时间步的输入除了正常的输入外还包括上一步的隐层输出)，对当下时间步的输出产生影响

<figure><img src="../.gitbook/assets/RNN2.gif" alt=""><figcaption></figcaption></figure>

从两个角度对RNN模型进行分类. 第一个角度是输入和输出的结构, 第二个角度是RNN的内部构造

* 按照输入和输出的结构进行分类:
  * N vs N - RNN
    * RNN最基础的结构形式，特点：**输入和输出序列是等长的**。由于这个限制的存在，使其适用范围比较小，可用于生成等长度的合辙诗句
  * N vs 1 - RNN
    * 要处理的问题输入是一个序列，而要求输出是一个单独的值而不是序列
    * 在最后一个隐层输出h上进行线性变换来实现，大部分情况下，为了更好的明确结果, 还要使用sigmoid或者softmax进行处理. 这种结构经常被应用在文本分类问题上.
  * 1 vs N - RNN
    * 如果输入不是序列而输出为序列的情况怎么处理呢？我们最常采用的一种方式就是使该输入作用于每次的输出之上. 这种结构可用于将图片生成文字任务等
  * N vs M - RNN
    * 这是一种不限输入输出长度的RNN结构, 它由编码器和解码器两部分组成, 两者的内部结构都是某类RNN, 它也被称为seq2seq架构. 输入数据首先通过编码器, 最终输出一个隐含变量c, 之后最常用的做法是使用这个隐含变量c作用在解码器进行解码的每一步上, 以保证输入信息被有效利用
    * seq2seq架构最早被提出应用于机器翻译, 因为其输入输出不受限制，如今也是应用最广的RNN模型结构. 在机器翻译, 阅读理解, 文本摘要等众多领域都进行了非常多的应用实践
    *

        <figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>



* 按照RNN的内部构造进行分类:
  * 传统RNN
  * LSTM
  * Bi-LSTM
  * GRU
  * Bi-GRU

#### RNN代码实战

nn.RNN类初始化参数

* input\_size:输入张量x中特征维度的大小
* hidden\_size:隐层张量h中特征维度的大小
* num\_layers:隐含层数量
* nolinearity:激活函数（默认是tanh）

nn.RNN实例化参数

* input：输入张量
* h0：初始化时的隐层张量h

```python
import torch
import torch.nn as nn
rnn = nn.RNN(5,6,1)
input = torch.randn(1,3,5) #生成一个大小为1x3x5的张量，5对应nn.RNN(5,6,1)中的5,3是batch size
h0 = torch.randn(1,3,6) # 6对应nn.RNN(5,6,1)中的6,3是batch size
output , hn = rnn(input,h0)
```

#### 传统RNN的优缺

* 内部结构简单，计算资源要求低，在短序列任务上性能和效果豆表现优异
* 传统RNN在解决长序列之间的关联时表现很差，在反向传播时会产生梯度消失（导致权重无法更新）或梯度爆炸（网络参数溢出，产生NaN值）
* 梯度消失产生的原因是比如因为使用sigmoid的导数值域在\[0,0.25]，在不断进行公式连乘之后，最后的梯度就会接近0

### LSTM模型

结构

* 遗忘门
  * $$f_t = σ(W_f*[h_t-1,x_t]+b_f)$$，将当前时间步输入与上一个时间步隐含状态h(t-1)拼接，得到\[x(t), h(t-1) ]，然后进行全连接变换，使用sigmoid进行激活
  * 根据当前时间步术后如和上一个时间步隐含状态h(t-1) 来决定遗忘多少上一层的细胞状态
* 输入门
* 细胞状态
  * 细胞状态更新图与计算公式：$$C_t = f_t*C_{t-1}+i_t*C_t$$，
* 输出门
  * 包含两个部分：（1）计算输出门的门值，和遗忘门/输入门的计算方式相同$$σ_t = σ(W_σ*[h_t-1,x_t]+b_σ)$$，（2）计算这个门值产生的隐含状态$$h_t = σ_t * tanh(C_t)$$，

<figure><img src="../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

#### LSTM代码实战

nn.LSTM初始化对象参数

* input\_size：输入张量x中特征维度的大小
* hidden\_size：隐层张量h中特征维度的大小
* num\_layers：隐含层数量
* bidirectional：是否选择使用双向LSTM，默认不使用

nn.LSTM实例化对象参数

* input：输入张量
* h0：初始化的隐层张量h
* c0：初始化的细胞状态张量c

```python
rnn = nn.LSTM(input_size,hidden_size,num_layers)
input = torch.randn(sequence_length,batch_size,input_size)
h0 = torch.randn(num_layers * num_directions,batch_size,hidden_size)
#num_directions非BiLSTM时为1
c0 = torch.randn(num_layers * num_directions,batch_size,hidden_size)
output,(hn,cn) = rnn(input,(h0,c0))
```

### Bi-LSTM模型

Bi-LSTM即双向LSTM，基本结构和LSTM相同，只是将LSTM应用两次，从左到右和从右到左传导，再将两次得到的LSTM结果进行拼接作为最终输出



### GRU模型

GRU（Gated Recurrent Unit），门控循环单元结构，同LSTM能够有效捕捉长序列之间的语义关联，环节梯度消失或爆炸现象，同时它的结构比LSTM更简单，结构：

* 更新门
* 重置门
*

    <figure><img src="../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>



nn.GRU类初始化主要参数

* input\_size: 输入张量x中特征维度的大小
* hidden\_size: 隐层张量h中特征维度的大小
* num\_layers: 隐含层的数量
* bidirectional: 是否选择使用双向LSTM，默认不使用

nn.GRU类实例化对象主要参数解释:

* input: 输入张量x.
* h0: 初始化的隐层张量h.

### TRM模型

输入——> Encoders ——> Decoders ——> 输出

n个结构完全相同的Encoders + n个完全相同的Decoders

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

Encoder分为三个部分：

1.  输入部分

    Embedding

    位置编码

    <figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>
2. 多头注意力机制
3. 前馈神经网络

### BERT模型

无监督的文本语义相似度，cls向量不能代表整个句子的语义信息

Input——>Token Embeddings——>Segment Embedings ——> Position Embedding

预训练：MLM+NSP

AR模型（自回归，只考虑单侧信息）和AE模型（自编码模型，使用上下文信息）
