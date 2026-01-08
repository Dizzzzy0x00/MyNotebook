# Crypto and application-1

## CH1 密码技术概述

#### 密码学学科特点&密码学研究的主要内容

是信息的机密性、完整性、鉴别和不可抵赖性等信息安全问题相关的一门学科

<figure><img src="../.gitbook/assets/0d7e1ab4-4ab9-47af-8c7b-7d8f4107e4ac.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

#### 密码系统的安全条件

理论不可破译和实际不可破译

Kerckhoffs原则（柯克霍夫斯原则）：一切秘密寓于密钥之中

* **密码体制即使不是理论上不可破译的，也应该是实际不可破译的**
* **密码体制的泄露不应该给保密通信者带来麻烦**

**一个提供机密性服务的密码系统是实际可用的**，必须满足的基本要求：

①系统的保密性不依赖于对加密体制或算法的保密，而仅依赖于密钥的安全性。 “一切秘密寓于密钥之中”是密码系统设计的一个重要原则。

②满足实际安全性，使破译者取得密文后在有效时间和成本范围内，确定密钥或相应明文在计算上是不可行的。

③加密和解密算法应适用于明文空间、密钥空间中的所有元素。

④加密和解密算法能有效地计算，密码系统易于实现和使用。

<figure><img src="../.gitbook/assets/3d25bcea-9a76-41a0-81c2-8034de1484b2.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/4dd7dcb4fd65a74ace539e2f3f0e43e3.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/3ed55115-8695-48b1-8967-e8370e8dc0d1.png" alt=""><figcaption></figcaption></figure>

#### 两种保密通信模型

<figure><img src="../.gitbook/assets/83e1b213-90bb-4a7e-b5e7-fcfd8a150b66.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/fcd8ddc4-eac4-419f-8b16-9bcff22335e5.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/266a2d91-626f-4524-9d66-219604c75505.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/270e1314-613c-45ca-997d-cf8c61d08636.png" alt=""><figcaption></figcaption></figure>

#### 对密码系统的两种攻击类型：主动攻击与被动攻击

所谓的被动式攻击意指入侵者（非法）**取得信息资产的存取权限，但是并未对其内容进行窜改**。主要的攻击方式有以下两种：

1. **窃听**（Eavesdropping）：窃听是指入侵者针对档案或通信内容进行监控。最常见的例子是于视频中时常出现的电话监听与网络监听等等。
2. **通信分析**（Traffic Analysis）：流量分析是针对网络通信的流量、内容、以及行为等等进行分析，透过通信内容或者流量的分析可以获得目标网络可观的数据，如：服务器位址、通信模式等等。

所谓的主动式攻击意指入侵者**针对档案或通信内容进行伪造或修改**，可能为以下四种攻击型式之一或是采取混合方式进行：

1. **伪装**（Masquerade）：伪装是指攻击者欺骗认证系统，非法取用系统资源。例如利用社交工程法骗取，或者利用网络窃听的方式取得密码后登入系统。
2. **回放**（Replay）：回放是指攻击者将从网络上截取的某些通信内容（如认证信息）重新发送，以欺骗服务器认证机制。常见的实例如早期Windows网络芳邻采用哈希方式（Hash Function）进行密码的加密，入侵者若能截取获得编码后的密码内容，可以利用重送一次的方式取得系统登入的授权。
3. **信息窜改**（Message Modification）：信息窜改指攻击者针对网络通信的内容进行删增或者更动。通信劫夺（Session Hijacking）利用TCP/ IP网络通信的弱点，抢夺合法使用者的通信频道，进而获得系统的操作权限，这种方式为信息窜改的一个实例。
4. **服务阻绝**（Denial of Service）：服务阻绝大概是大家最耳熟能详的攻击方式，攻击者透过各种可能的方法（ICMP flooding、SYN Flooding、Mail Bomb）等等方式使得使用者与管理者无法取得系统资源及服务。

#### 密码分析攻击的主要类型：

根据密码分析者对明文、密文等数据资源的掌握程度可以分为四类分析攻击：

<figure><img src="../.gitbook/assets/dafa59e3-785a-4b7c-aec2-2c8e8da2cccf.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/a1a30d1e-0896-4f97-be39-41e3d2d3f1fa.png" alt=""><figcaption></figcaption></figure>

#### 密码算法的分类

*

    <figure><img src="../.gitbook/assets/6bca2538-6929-40b5-b235-ce7e7a96862e.png" alt=""><figcaption></figcaption></figure>
*

    <figure><img src="../.gitbook/assets/59b9cbc7-1807-44a6-9abc-057cb7807255.png" alt=""><figcaption></figcaption></figure>
*   &#x20;

    <div data-full-width="true"><figure><img src="../.gitbook/assets/3f085091-74ee-4610-9a5e-78d6f38c6e38.png" alt=""><figcaption></figcaption></figure></div>

    <figure><img src="../.gitbook/assets/290f726a-1391-42b8-8b7d-d635b08102bc.png" alt=""><figcaption></figcaption></figure>
*

    <figure><img src="../.gitbook/assets/3cb84ff9-6ebe-4ed7-ba7e-1462a6ffd3f0.png" alt=""><figcaption></figcaption></figure>



#### 确定性和非确定性密码算法

**确定性密码**：在给定**相同的密钥和相同的明文**时，算法每次运行都会产生**完全相同的密文**，整个过程不引入随机因素

<p align="center"><span class="math">C = Enc(K, M)</span></p>

**（1）无随机性：**&#x7B97;法不使用随机数或随机填充，输出结果完全由输入决定。

**（2）可重复性强：**&#x540C;一输入在任何环境下加密结果一致，便于调试、测试和结果验证。

**（3）结构简单，实现成本低：**&#x65E0;需安全随机数生成器（CSPRNG），实现复杂度相对较低。

**（4）安全性依赖密钥而非随机性：**&#x5728;现代安全模型下，其安全性通常较弱，容易泄露明文模式，

* **明文模式泄露**：相同明文 → 相同密文
* **易受字典攻击、频率分析攻击**
* **通常不满足 IND-CPA（选择明文攻击下的不可区分性）**

例如：（1）古典密码（凯撒密码、维吉尼亚密码）（2）分组密码的 ECB 模式（如 AES-ECB）（3）确定性哈希（在固定输入下输出固定）

非确定性密码算法（**概率密码算法）：概率密码算法**在加密过程中引入**随机性**，即使在使用相同密钥和相同明文的情况下，不同加密过程也可能产生**不同的密文：**

$$
C=Enc(K,M;r)
$$

引入随机性（Nonce / IV / Salt）；安全性更高，但实现更复杂；**满足现代安全定义，**&#x901A;常可满足：IND-CPA，在认证加密中甚至满足 IND-CCA

* 防止重放分析
* 防止字典攻击
* 防止流量模式泄露
* 可抵御大多数已知的被动攻击模型

例如：（1）RSA-OAEP（概率公钥加密）（2）ElGamal 加密（3）AES-CBC（使用随机 IV）（4）带随机 salt 的密码哈希（如 bcrypt、scrypt）

#### 对称密码体制和非对称密码体制

<figure><img src="../.gitbook/assets/24e2ee22-eaf7-46bc-b2ce-9b91f83bdf2f (1).png" alt=""><figcaption></figcaption></figure>

#### 针对不同功能密码算法的攻击目标

<figure><img src="../.gitbook/assets/537bdfdd-c4ba-432c-897e-fe36527550cc.png" alt=""><figcaption></figcaption></figure>

| 密码功能      | 核心安全目标    | 攻击目标             |
| --------- | --------- | ---------------- |
| 对称/公钥加密   | 机密性       | 明文恢复 / 密钥恢复 / 区分 |
| MAC       | 完整性       | 伪造合法 MAC         |
| 哈希函数      | 抗碰撞 / 抗原像 | 原像 / 二次原像 / 碰撞   |
| 数字签名      | 不可否认      | 伪造签名             |
| 密钥交换      | 安全建密      | 会话密钥恢复 / MITM    |
| PRG / PRF | 不可区分      | 区分真随机            |



## CH1-2 古典密码

#### 替换密码和置换密码

替换密码：

**（1）单表替换密码（Monoalphabetic）**

* 同一个明文字母始终被替换为同一个密文字母
* 例如：凯撒密码 $$c=(p+k) mod 26$$

**（2）多表替换密码（Polyalphabetic）**

* 同一明文字母在不同位置可能被替换为不同密文字母
* 例如：维吉尼亚密码

<figure><img src="../.gitbook/assets/31c58336-f702-4fe0-b232-039aaa9b53fb.png" alt="" width="563"><figcaption></figcaption></figure>

<div data-full-width="true"><figure><img src="../.gitbook/assets/fa4b8085-ebd4-40ba-a0b9-01e2e77a9dd1.png" alt="" width="563"><figcaption></figcaption></figure></div>

置换密码：

* 栅栏密码（Rail Fence Cipher）
* 列置换密码（Columnar Transposition Cipher）

<figure><img src="../.gitbook/assets/a8af2de8-dc86-4056-b289-731c155f0797.png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/e1794591-cca7-4ed6-9c27-857aeb210200.png" alt="" width="563"><figcaption></figcaption></figure>

| 对比维度    | 替换密码   | 置换密码 |
| ------- | ------ | ---- |
| 操作对象    | 符号     | 位置   |
| 是否改变字符  | 是      | 否    |
| 是否改变顺序  | 否      | 是    |
| 是否保持频率  | 是（单表）  | 是    |
| 抗统计攻击能力 | 弱      | 弱    |
| 密钥本质    | 映射（双射） | 排列   |

现代分组密码并不单独使用其中之一，而是通过**反复组合**：

* **替换（Substitution）** → 提供混淆（Confusion）
* **置换（Permutation）** → 提供扩散（Diffusion）

这正是 **Shannon 提出的“混淆与扩散原则”**，也是 SPN（Substitution–Permutation Network）结构的理论基础。

#### 凯撒密码、仿射密码、单表替换密码、多表替换密码的概念与特点

* 凯撒密码（k=3时的一般单表替代密码）
  * $$\mathrm{E}=\left\{E: Z_{26} \rightarrow Z_{26}, E_{\mathrm{k}}(m)=m+k(\bmod 26) \mid m \in M, k \in K\right\}$$
  * $$\mathrm{D}=\left\{D: Z_{26} \rightarrow Z_{26}, D_{\mathrm{k}}(m)=c-k(\bmod 26) \mid c \in C, k \in K\right\}$$
  * 密钥空间极小（仅 25 种有效密钥）
  * 完全保持明文字母的频率分布
  * 极易被穷举攻击和频率分析破解
  * 仅具教学和历史意义
* 仿射密码（Affine Cipher）
  * $$c= E_ {k}  (m)= k_{1}m+ k_ {2} (mod26)$$
  * $$m=D_k\left(c\right)=k_1^{-1}\left(c\text{一}k_2\right)\left(\mathrm{mod~}26\right)$$
  * 密钥空间（ $$K=\{(k_1,k_2)|k_1,k_2\in Z_{26},\gcd(k_1,26){=}1\}$$）比凯撒密码大，但仍有限
  * 保持明文字母频率分布
  * 易受已知明文攻击与频率分析
  * 线性结构明显，安全性较弱
* 单表替换密码
  * 整个加密过程中使用**同一张固定的替换表**，将每个明文字母替换为唯一的密文字母
  * 密钥空间为 26!，理论上很大
  * 明文字母与密文字母存在固定一一对应关系
  * 严格保持字母频率和统计特征
  * 在自然语言下可被频率分析、模式分析破解
* 多表替换密码
  * 多表替换密码在加密过程中**周期性或非周期性地切换多张替换表**，使同一明文字母在不同位置可能映射为不同密文字母
  * 显著削弱单一字母频率特征
  * 抗频率分析能力强于单表替换密码
  * 安全性依赖密钥长度与使用方式
  * 仍可通过 Kasiski 检验、Friedman 检验等方法破解

#### 维吉利亚(Vigenère)、 Playfair算法计算方法和特性

<div><figure><img src="../.gitbook/assets/b4fb3fa1-71e0-4ffc-a561-1863131dae37.png" alt="" width="375"><figcaption></figcaption></figure> <figure><img src="../.gitbook/assets/bd6fe8e2-7e8f-4e7b-9dd7-7a7e4b078403.png" alt="" width="375"><figcaption></figcaption></figure></div>

<div><figure><img src="../.gitbook/assets/2b48eb49-3794-4798-a9be-6a70d6812125.png" alt=""><figcaption></figcaption></figure> <figure><img src="../.gitbook/assets/72ebe9e1-cee0-418d-8d97-9007420b60e3.png" alt=""><figcaption></figcaption></figure></div>

<div><figure><img src="../.gitbook/assets/74c8a284-dad9-405a-b9a2-ee5046672165.png" alt=""><figcaption></figcaption></figure> <figure><img src="../.gitbook/assets/27d736b2-a54b-4a2d-bc08-a0353ece32eb.png" alt=""><figcaption></figcaption></figure></div>

<figure><img src="../.gitbook/assets/5a116aa8-deec-4708-9bde-8d605e2880e1.png" alt=""><figcaption></figcaption></figure>

#### Hill密码的特性与计算方法

<div><figure><img src="../.gitbook/assets/1ed8b17d-4cab-4490-a8c1-7e9d53cb2575.png" alt=""><figcaption></figcaption></figure> <figure><img src="../.gitbook/assets/a1b64adc-12d9-4bde-9d17-4517db713cb6.png" alt=""><figcaption></figcaption></figure></div>

<div><figure><img src="../.gitbook/assets/4e767151-523f-4114-b4f9-f568dcd717bb.png" alt=""><figcaption></figcaption></figure> <figure><img src="../.gitbook/assets/378442fa-0234-44b9-81f1-3211502aa0ef.png" alt=""><figcaption></figcaption></figure></div>

<div><figure><img src="../.gitbook/assets/3cf610ed-5ea3-483a-9b38-194488988fb3.png" alt=""><figcaption></figcaption></figure> <figure><img src="../.gitbook/assets/1aff6f7d-d941-4c87-870e-88361110eee4.png" alt=""><figcaption></figcaption></figure></div>

#### 转轮机密码、OTP等古典密码算法的原理与特点

**转轮机密码：**&#x8F6C;轮机密码是一类**机电式多表替换密码系统**，其核心思想是通过**多个可旋转的转轮（Rotor）构造一个动态变化的替换表**。

每个转轮本质上是一个固定的单表替换，但随着每次按键操作，转轮发生转动，从而改变整体的替换关系。一个**随时间变化的多表替换密码；**&#x5178;型代表：**Enigma 密码机**。

工作机制

* 明文输入一个字母
* 信号依次通过多个转轮（正向替换）
* 经反射器（Reflector）反射
* 再反向通过转轮
* 输出密文字母
* 每加密一个字符，至少有一个转轮转动

主要特点

* 同一明文字母在不同位置会被映射为不同密文字母
* 具备较强的混淆能力，远强于传统多表替换密码
* 密钥空间较大（转轮选择、初始位置、插线板设置）
* 仍存在结构性弱点（如自反性、不映射为自身等）
* 可被利用操作失误和统计规律进行系统性破译

**一次一密** One-Time Pad, OTP

**基本原理：**&#x4E00;次一密是一种基于**模加或异或运算**的流加密体制，其核心思想是：**使用一段真正随机、与明文等长、且只使用一次的密钥进行加密。**

常见形式（以比特串为例）：

加密： $$c = p \oplus k$$

解密： $$p = c \oplus k$$

**理论安全性：**&#x4F;TP 在严格满足以下条件时，具备**信息论安全性（Perfect Secrecy）**：

1. 密钥完全随机
2. 密钥长度 ≥ 明文长度
3. 密钥只使用一次
4. 密钥绝对保密

这是**唯一被严格证明具有完全保密性的加密方案**（Shannon 定理）。

**主要特点**

* 在理论上不可破解（即使无限计算能力）
* 不泄露任何关于明文的统计信息
* 密钥管理成本极高
* 一旦密钥复用，安全性彻底失效
* 实际系统中难以大规模部署

## CH2 香农保密理论基础

#### 密码体制的数学模型

<div><figure><img src="../.gitbook/assets/84096244-33f8-4c59-a8ac-fe24e7b89638.png" alt=""><figcaption></figcaption></figure> <figure><img src="../.gitbook/assets/27974c17-c142-4c6e-9312-6388c5b970ce.png" alt=""><figcaption></figcaption></figure></div>

#### 熵的概念与熵的特性

<div><figure><img src="../.gitbook/assets/976b0a9b-c492-4ede-946b-f78d3f4f14ce.png" alt=""><figcaption></figcaption></figure> <figure><img src="../.gitbook/assets/9d0cc579-8894-42ad-9143-6aa7894ab3e6.png" alt=""><figcaption></figcaption></figure></div>

<div><figure><img src="../.gitbook/assets/86db5e66-663f-4e81-8c9a-e2dca903887b.png" alt=""><figcaption></figcaption></figure> <figure><img src="../.gitbook/assets/image (103).png" alt=""><figcaption></figcaption></figure></div>

#### 多余度和唯一解距离

<figure><img src="../.gitbook/assets/19df0ea7-371e-418a-a062-fd781bcbc532.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/6002c289-9e97-4f07-bf9d-2418c31ac505.png" alt=""><figcaption></figcaption></figure>

当密码分析者所截获的密文字符数小于 $$U_d$$时，存在多个可能的解（存在伪密钥），但当所截获的密文字符数大于 $$U_d$$时，这种密码的破译问题理论上就存在唯一解。

**唯一解距离**表示了密码分析者必须处理的密文量的**理论下限，**&#x4F8B;如计算出密码的唯一解距离 $$U_d=28$$,则至少要给出长度为28的密文串，唯一地破解该密码才有可能

#### 完全保密体制和理论保密体制

完全保密性（Perfect Secrecy ） ：对于密码分析者通过观察密文不能获得有关明文的任何信息。\
定义： $$对\forall m\in M,c\in C,有:P_r(m|c)=P_r(m)$$即**明文m的后验概率等于明文m的**\
**先验概率**，则该密码体制具有完全保密性。

密码体制的完全保密性是**针对唯密文攻击而言的**。一个完全保密密码体制并**不能保证它在已知明文攻击或选择明文攻击下也是安全的**。

<figure><img src="../.gitbook/assets/ecc2d41d-4914-4ee5-9215-e4154448c5b4.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/5f5b56b7-9439-4fb3-8e7d-84fcb387b132.png" alt=""><figcaption></figcaption></figure>

理想保密性（Ideal Secrecy ）：\
定义：当一个密码体制的**唯一解距离** $$U_d$$**趋向于无穷大**时，该密码体制就称为具有理想保密性。

<figure><img src="../.gitbook/assets/f7119247-a0a2-4633-bd7e-8cc1735dd844.png" alt=""><figcaption></figcaption></figure>

#### 扩散与扰乱原理&#xD;

扩散原理(diffusion)：尽量使得明文的每一位都影响密文中许多位的值\
扰乱原理(confusion)：使密文与明文、加密密钥之间的统计关系尽量复杂化，使破译者难以从密文的统计特性导出与密钥相关的信息

#### 乘积密码体制

对于幂等密码，有:

$$
对 m\in M,\forall k_1,k_2\in K,\exists k_3\in K,
满足：E_{k_2}(E_{k_1}(m))=E_{k_3}(m)
$$

**迭代密码体制必须使用非幂等密码体制。**

<div><figure><img src="../.gitbook/assets/0393ae9a-a272-4a62-b936-3a274d75a505.png" alt=""><figcaption></figcaption></figure> <figure><img src="../.gitbook/assets/1c99b68b-7244-4f11-b6b5-1adefb2074d8.png" alt=""><figcaption></figcaption></figure></div>

## CH-3 对称密码技术

#### 分组密码的设计原理和整体结构

分组密码是一种将明文分成固定长度的块，然后对每个块进行加密的密码算法。在**设计分组密码时，需要满足以下要求**：

1. **分组长度足够大**：为了提高安全性，分组长度应该足够大。当明文分组长度为n位时，至多需要2n个明文密文对就可完全破解密码。因此，分组长度的大小应该足够大，**防止攻击者通过收集不同的密文、明文对来进行穷举明文空间攻击**
2. **密钥空间足够大**：**防止攻击者穷举密钥空间的攻击**
3. **密码算法足够复杂**：分组密码的设计要求密码变换必须足够复杂，使攻击者**除了强行攻击（穷举）外，找不到其他便捷的数学破译方法**。这样可以有效防止攻击者通过分析密文得到有意义的明文。

分组密码的设计原理主要包括**数学规则、简单函数和非线性函数等运算**。

在整体结构上，分组密码通常包括**密钥调度、初始置换、多次轮函数迭代和最后的逆初始置换**等部分。其中，**轮函数**是分组密码的核心部分，由非线性函数构成，可以有效地防止攻击者通过分析密文得到明文。

分组密码的分析方法主要包括朴素密码分析、差分密码分析、线性密码分析和相关密钥密码分析等。这些方法都是通过分析密文和密钥之间的关系，尝试破解出明文和密钥。为了提高安全性，现代的分组密码算法通常采用多种分析方法的结合，以增加破解的难度。

密钥管理是分组密码应用中的重要环节。为了保证安全性，密钥应该在安全的环境中生成、存储和使用。同时，应该定期更换密钥，以降低密钥被破解的风险。此外，对于一些重要的数据加密场景，可以采用一些额外的安全措施，如使用硬件加密模块等来保护密钥的安全性。

分组密码的工作模式主要包括**电子密码本模式、密文链接模式和计数器模式**等。这些模式可以根据实际应用场景选择使用。在选择工作模式时，应该充分考虑安全性、效率和实现难度等因素。

分组密码的检测与评估：为了确保分组密码的安全性，需要对算法进行严格的检测和评估。评估指标主要包括算法的安全性、效率、实现难度和资源占用等方面。此外，还可以通过对比分析不同的分组密码算法来评估其优劣

#### 分组密码的两种基本结构SPN，Feistel、分组密码原理与概念

<div><figure><img src="../.gitbook/assets/113de309917114c7b80aad0668292949.png" alt=""><figcaption></figcaption></figure> <figure><img src="../.gitbook/assets/624431ab-4a31-4e0a-91bd-fe98bfe33f52.png" alt=""><figcaption></figcaption></figure></div>

<figure><img src="../.gitbook/assets/891a008a-610e-49fb-8d33-16017b069e41.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/a356f335-f179-4ee3-8b5a-12983e854046.png" alt=""><figcaption></figcaption></figure>

<div><figure><img src="../.gitbook/assets/57589dad-04b1-4121-bf9d-a3561bc8b83c.png" alt=""><figcaption></figcaption></figure> <figure><img src="../.gitbook/assets/321f45ac-9e23-4e7e-a0a0-4cd09e71f219.png" alt=""><figcaption></figcaption></figure></div>

最后的解密输出为： ，即加密输入的明文分组
