---
description: AFL白皮书阅读笔记
---

# AFL Whitepaper

{% embed url="https://lcamtuf.coredump.cx/afl/technical_details.txt" %}

### Part0 AFL工作流程

1. 从源码编译程序时进行插桩，以记录代码覆盖率（Code Coverage）；
2. 选择一些输入文件，作为初始测试集加入输入队列（queue）；
3. 将队列中的文件按一定的策略进行“突变”；
4. 如果经过变异文件更新了覆盖范围，则将其保留添加到队列中;
5. 上述过程会一直循环进行，期间触发了crash的文件会被记录下来。

<figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

### Part1 覆盖率度量

AFL采用基于分支(边缘)覆盖率的度量方式——_“branch (edge) coverage”_

注入到编译程序中的检测程序会捕获边缘覆盖率以及粗略的边缘命中计数，在分支点进行插桩(instrumentation)

```
  cur_location = <COMPILE_TIME_RANDOM>;
  shared_mem[cur_location ^ prev_location]++;
  prev_location = cur_location >> 1;
  //右移操作就是为了让tuples有定向性，即区分路径AB和BA
```

可以注意到 cur\_location 是由TIME\_RANDOM随机生成，有助于：

1. 简化链接复杂项目过程的复杂度
2. 保持 XOR（二进制异或） 输出的均匀分布。

_**理解这里的随机生成**：每一个独立的代码块或路径，在编译时都会分配到一个随机值。这个随机值作为位置标记，用来表示控制流程中的一个独特点。所以虽然 `cur_location` 实际上是随机的，但它还是会被用来标识特定的代码位置或路径。_

`shared_mem[]` 数组是一个 64 kB 共享内存，位图中每个字节都可以被认为是对被测程序中特定 (branch\_src, branch\_dst) **元组**的命中，即`shared_mem[]`记录了AFL检测到的**粗略**的分支执行命中次数(branch-taken hit counts)，这里提到的元组就是之前提到的分支路径，例如下面两个路径对应的元组转换：

```
A -> B -> C -> D -> E (tuples: AB, BC, CD, DE)
A -> B -> D -> C -> E (tuples: AB, BD, DC, CE)
```

{% hint style="info" %}
大小设定为64KB是平衡测试速度和碰撞率最后做出的折中，很多程序的边缘数在 2k 到 10k 可之间，同时它的大小足够小，允许接收端在几微秒内分析位图，匹配 L2 缓存速度。
{% endhint %}

### Part2 检测新路径

根据前面Part1中提到的：AFL通过输入文件的突变产生新路径(新元组tuples)，当一个变异的输入产生了一个包含**新tuple的执行路径**时，对应的输入文件就被**保存**，这种变异测试用例会被加入到输入队列(input queue)中，当做下一次fuzz的起点。对于那些**没有产生新路径的输入**，就算他们的路径是不同的，也会被**抛弃**掉。

看一个例子：

```
#1: A -> B -> C -> D -> E
#2: A -> B -> C -> A -> E
```

input 2相对于input 1出现了新的tuples(CA, AE)，所以这个测试用例被保存，加入到输入队列(input queue)，但是此时如果一条新的路径出现时：

```
#3: A -> B -> C -> A -> B -> C -> A -> B -> C -> D -> E
```

尽管input 3的执行路径在整体上看非常不同，但AFL认为他们是同一条路径，因为只出现了AB BC CD DE CA AE这几个元组，并没有新的路径出现。

这种方法允许对程序状态进行非常细粒度和长期的探索，同时不必对复杂的执行轨迹执行任何计算密集型和脆弱的全局比较，同时避免路径爆炸

AFL的fuzzer也会粗略地记录tuple的**命中数(hit counts)**。这些被分割成几个buckets：

```undefined
1, 2, 3, 4-7, 8-15, 16-31, 32-127, 128+
```

{% hint style="info" %}
_为什么要这样子划分命中次数？这样子划分允许将插桩生成的 8 位计数器就地映射到模糊器可执行文件所依赖的 8 位位图，以跟踪已经看到的每个元组的执行计数_
{% endhint %}

**AFL认为，从一个bucket变成另一个bucket才是重要的（**会被标记被程序流中的一个有趣的变化interesting change**）**，而忽略单个桶范围内的变化



通过命中次数(hit count)，我们能够分辨控制流是否发生变化。例如一个代码块被执行了两次，但只命中了一次。并且这种方法对循环的次数不敏感(循环47次和48次没区别)。



### Part3 输入队列的进化

变异测试用例(Mutated test cases)是能够产生新路径(即新的tuple)的测试用例。这种变异测试用例会被加入到输入队列(input queue)中，当做下一次fuzz的起点。它们作为已有测试用例的补充，但并不替换掉已有测试用例。

与更贪婪的遗传算法相比，这种方法允许工具逐步探索底层数据格式的各种不相交和可能相互不兼容的特征，如下图所示:

<figure><img src="../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://lcamtuf.blogspot.com/2014/11/pulling-jpegs-out-of-thin-air.html" %}
由只包含“hello”的文本fuzzer模糊生成JPEG文件，并用于djpeg的测试
{% endembed %}

blind fuzzing（盲测）生成测试数据的时候不考虑数据的质量，通过大量测试数据来概率性地触发漏洞。Guided fuzzing（有引导的测试）则关注测试数据的质量，期望生成更有效的测试数据来触发漏洞的概率。AFL就是通过测试覆盖率来衡量测试输入的质量，希望生成有更高测试覆盖率的数据，从而提升触发漏洞的概率。

### Part4  种子库筛选&#x20;

随着fuzz的进行，会不断进化并生成许多新的种子加入到输入队列中，但是一些新的种子可能会覆盖它们的祖先用例已经提供了的覆盖点，也就是后来的测试用例可能提供了更多的覆盖范围。所以很有必要对种子库进行一个简单的筛选，优化后续的模糊测试

为了优化模糊测试工作，AFL 使用一个**快速算法周期性的重新评估(re-evaluates)队列**，这个算法选择队列的一个更小的子集，并且这个子集仍能覆盖所有的tuple。

该算法的工作原理是为每个队列条目分配一个与其执行延迟和文件大小成正比的分数；然后为每个元组选择得分最低的候选者

AFL使用简单的工作流程顺序处理元组：

1. 在临时工作集中查找下一个元组，
2. 找到这个元组的获胜队列条目，
3. 在工作集中注册该条目的跟踪中出现的所有元组，
4. 如果在集合中有任何缺失的元组，则转到#1。

生成的“收藏”条目的语料库通常比起始数据集小5-10倍。非优先项**不会被丢弃**，但是在队列中遇到时，它们会**以不同的概率被跳过，**可以看出AFL的**语料库筛选是基于经验的：**

* 如果种子库中有新的、尚未模糊的收藏种子，99% 的非收藏种子将被跳过以到达收藏种子。
* 如果没有新的收藏种子：
  * 如果当前不受欢迎的种子之前被 fuzz，它将被跳过的概率为 95%&#x20;
  * 如果它还没有经过任何模糊测试，跳过的几率会下降到 75%。



### Part5 种子库修剪

文件大小对模糊测试性能有显着影响，因为大文件会使目标二进制文件变慢，而且因为它们降低了突变重要控制结构而不是冗余数据块的可能性。

AFL**使用变化的长度和步距来连续地删除数据块**；**任何不影响位图跟踪校验和的删除块将被保存下来**。这个修剪器的设计并不算特别地周密，相反地，它试着在精确度和进程调用的次数之间选取一个平衡，找到一个合适的块大小和步长。平均每个文件将增大约5-20%。





### 语料库蒸馏（Corpus Distillation）

AFL提供工具afl-cmin和afl-tmin

* AFL-CMIN：用来移除执行相同代码的输入文件
  * 尝试找到与语料库全集具有相同覆盖范围的最小子集
  * 能够对输入或输出的语料库进行稍微复杂但慢得多的的处理， 可以精简语料库，去掉可能重复的测试用例，针对一些复杂的语料库十分有用，可大大减少无用的 fuzz 用例。
* AFL-TMIN：用来减小单个输入文件的大小
  * 减小**单个文件**的大小，对每个文件进行更细化的处理，因为 afl 要求测试用例的大小最好小于 1KB，因此最好将精简后的用例进一步缩小体积。
  * afl-tmin接受单个文件输入，所以可以用一条简单的shell脚本批量处理。如果语料库中文件数量特别多，且体积特别大的情况下，这个过程可能花费几天甚至更长的时间
  * 实际的最小化算法是：
    1. 尝试使用大步长的方式对大的块进行归零处理。经验表明，可通过稍后更细粒度的工作来减少执行次数。
    2. 以二进制搜索方式执行块删除过程，减少块大小和步长。
    3. 通过计算唯一性字符，以及尝试使用0值进行批量替换，达到进行字母标准化的目的。
    4. 最后，对非零字节执行逐字节规范化。



### Part6 变异策略

AFL使用的变异策略——format-agnostic，详细的细节参考开发者的文章：

{% embed url="http://lcamtuf.blogspot.com/2014/08/binary-fuzzing-strategies-what-works.html" %}
_**“为Fuzz设计突变引擎更多的是艺术而不是科学”**_
{% endembed %}

确定性策略包括：

* **bitflip，位反转**
  * 即按位进行翻转，0变1，1变0。
  * 根据**翻转量/步长**，按位翻转的策略有以下几种&#x20;
    * bitflip 1/1，每次翻转1个bit，按照每1个bit的步长从头开始
    * bitflip 2/1，每次翻转相邻的2个bit，按照每1个bit的步长从头开始
    * bitflip 4/1，每次翻转相邻的4个bit，按照每1个bit的步长从头开始&#x20;
    * bitflip 8/8，每次翻转相邻的8个bit，按照每8个bit的步长从头开始，即依次对每个byte做翻转
    * &#x20;bitflip 16/8，每次翻转相邻的16个bit，按照每8个bit的步长从头开始，即依次对每个word做翻转
    * &#x20;bitflip 32/8，每次翻转相邻的32个bit，按照每8个bit的步长从头开始，即依次对每个dword做翻转
* **aarithmetic，进行加减操作**
  * arith 8/8，每次8bit进行加减运算，8bit步长从头开始，即对每个byte进行整数加减变异；
  * &#x20;arith 16/8，每次16bit进行加减运算，8bit步长从头开始，即对每个word进行整数加减变异；&#x20;
  * arith 32/8，每次32bit进行加减运算，8bit步长从头开始，即对每个dword进行整数加减变异；
* **interest**
  * 把一些“有意思”的特殊内容替换到原文件中， interest的三个步骤跟arithmetic相同
    * &#x20;interest 8/8，每次8bit进行加减运算，8bit步长从头开始，即对每个byte进行替换；&#x20;
    * interest 16/8，每次16bit进行加减运算，8bit步长从头开始，即对每个word进行替换；
    * &#x20;interest 32/8，每次32bit进行加减运算，8bit步长从头开始，即对每个dword进行替换；
  *

      <pre class="language-cpp"><code class="lang-cpp"><strong>#define INTERESTING_8                                    \
      </strong>  -128,    /* Overflow signed 8-bit when decremented  边界溢出条件*/ \
            -1,  /*                                         */ \
            0,   /*                                         */ \
            1,   /*                                         */ \
            16,  /* One-off with common buffer size         */ \
            32,  /* One-off with common buffer size         */ \
            64,  /* One-off with common buffer size         */ \
            100, /* One-off with common buffer size         */ \
            127                        /* Overflow signed 8-bit when incremented  */
            
      #define INTERESTING_16                                    \
        -32768,   /* Overflow signed 16-bit when decremented */ \
            -129, /* Overflow signed 8-bit                   */ \
            128,  /* Overflow signed 8-bit                   */ \
            255,  /* Overflow unsig 8-bit when incremented   */ \
            256,  /* Overflow unsig 8-bit                    */ \
            512,  /* One-off with common buffer size         */ \
            1000, /* One-off with common buffer size         */ \
            1024, /* One-off with common buffer size         */ \
            4096, /* One-off with common buffer size         */ \
            32767                      /* Overflow signed 16-bit when incremented */
            
      #define INTERESTING_32                                          \
        -2147483648LL,  /* Overflow signed 32-bit when decremented */ \
            -100663046, /* Large negative number (endian-agnostic) */ \
            -32769,     /* Overflow signed 16-bit                  */ \
            32768,      /* Overflow signed 16-bit                  */ \
            65535,      /* Overflow unsig 16-bit when incremented  */ \
            65536,      /* Overflow unsig 16 bit                   */ \
            100663045,  /* Large positive number (endian-agnostic) */ \
            2147483647                 /* Overflow signed 32-bit when incremented */

      </code></pre>


* **dictionary**
  * 用户提供的字典里有token，用来替换要进行变异的文件内容，如果用户没提供就使用 bitflip 自动生成的 token。

非确定性策略的步骤包括：

* （havoc）多位翻转、插入、删除、算数加减等
* （splice）不同测试用例之间的随机拼接。

早期模糊阶段，afl-fuzz 所做的大部分工作实际上是高度确定性的，并且在后期才发展到 havoc （随机大破坏）和 splice （随机拼接），这里我认为**Havoc和Splice才是AFL模糊器的精髓**所在，关于Havoc，强烈推荐FuzzWiki中相关的技术解析：

<figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://mp.weixin.qq.com/s/A2Dk6WOZo7FZ_3bFRSsk-Q" %}

### Part7 字典

插桩提供的反馈能够让它自动地识别出一些输入文件中的语法tokens，并且能够为测试器检测到一些组合，这些组合是由预定义的或自动检测到的字典项(dictionary terms)构成的合法语法(valid grammar)。

当基本的、通常容易获得的语法标记以纯粹随机的方式组合在一起时，插桩和队列进化这两种方法共同提供了一种反馈机制，这种反馈机制能够区分无意义的变异和在插桩代码中触发新行为的变异。这样能增量地构建更复杂的句法。这样构建的字典能够让fuzzer快速地重构非常详细且复杂的语法，比如JavaScript, SQL,XML。



### Part8 崩溃去重

对于任何有效的fuzzing工具来说，重复崩溃是问题之一，Fuzzer需要设计对崩溃进行去重的策略。在AFL中，如果满足以下两个条件中的任何一个，afl-fuzz 中实现的解决方案会认为崩溃是唯一的：

* 这个崩溃的元组集合包括一个之前崩溃从未见到过的tuple，
* 崩溃的元组集合缺少一个始终存在于早期崩溃中的元组。

这种方式一开始容易受到数值膨胀的影响，但实验表明其有很强的自我限制效果。



### Part9 崩溃调查

在使用崩溃去重策略筛选出唯一的crash以后，需要对崩溃进行调查，这是因为不同的crash的可用性(exploitability)是不同的。afl-fuzz提供一个crash的探索模式(exploration mode)来解决这个问题。在这种模式下，对一个已知的出错测试用例，它被fuzz的方式和正常fuzz的操作没什么不同，但是不能导致崩溃的变异结果会被丢弃。这种方法使用插桩反馈的方式探索崩溃程序的状态，目的是通过不明确的错误条件，然后隔离出新发现的测试输入提供给人工复查。

意义参考文章：

{% embed url="https://lcamtuf.blogspot.com/2014/11/afl-fuzz-crash-exploration-mode.html" %}



### Part10 Fork serve机制

为了提升性能，afl-fuzz使用了一个“fork server”，fuzz进程只进行一次execve(),linking和libc initialization，之后的fuzz进程通过写时拷贝技术从已经停止的fuzz进程镜像直接拷贝。

### Part11 并行机制

并行化机制依赖于定期检查由其他 CPU 内核或远程机器上独立运行的实例产生的队列，然后有选择地拉入测试用例，当在本地尝试时，会产生当前模糊器尚未看到的行为。

这使得模糊器设置具有极大的灵活性，包括针对通用数据格式的不同解析器运行同步实例，通常具有协同效应。

### Part12 二进制instrumentation <a href="#id-12-er-jin-zhi-instrumentation" id="id-12-er-jin-zhi-instrumentation"></a>

在“用户模拟”模式下，借助单独构建的 QEMU 版本，可以完成黑盒、仅二进制目标的检测。这也允许跨架构代码的执行 - 例如，x86 上的 ARM 二进制文件。

QEMU 使用基本块作为翻译单元；检测是在此之上实现的，并使用与编译时hooks大致类似的模型：

```
  if (block_address > elf_text_start && block_address < elf_text_end) {
​
    cur_location = (block_address >> 4) ^ (block_address << 8);
    shared_mem[cur_location ^ prev_location]++;
    prev_location = cur_location >> 1;
​
  }
```

第二行中基于移位和异或的加扰用于屏蔽指令对齐的影响。

QEMU、DynamoRIO 和 PIN 等二进制转换器的启动相当慢；为了解决这个问题，QEMU 模式利用了一个类似于用于编译器检测代码的 fork server，过把一个已经初始化好的进程镜像，直接拷贝到新的进程中。

新基本块的首次转换也会导致大量延迟。为了消除这个问题，AFL fork server通过在运行的模拟器和父进程之间提供一个通道。该通道用于将任何新遇到的块的地址通知给父进程，并将它们添加到缓存中，以便直接复制到将来的子进程中。

由于这两个优化，QEMU 模式的开销大约是慢 2-5 倍，而 PIN 的开销是慢 100 倍以上。

### **Part13 The afl-analyze tool**

文件格式分析器是最小化算法的简单扩展；该工具不是试图删除无操作块，而是执行一系列行走字节翻转，然后在输入文件中对字节的运行进行注释。

它采用以下分类方案：

* “_无操作块/No-op blocks_” - 位翻转不会导致控制流发生明显变化的段。常见的例子可能是注释部分、位图文件中的像素数据等。
* “_表面内容/Superficial content_” - 部分（但不是全部）bitflip 产生一些控制流变化的片段。示例可能包括富文档（例如 XML、RTF）中的字符串。
* “_关键流/Critical stream_” - 一个字节序列，其中所有位翻转以不同但相关的方式改变控制流。这可能是压缩的数据，非原子比较的关键字或魔术值，等等。
* “_可疑长度字段/Suspected length field_” - 小的原子整数，当以任何方式触摸时，会导致程序控制流的一致更改，提示长度检查失败。
* “_可疑的校验和或魔法 int/Suspected cksum or magic int_” - 一个整数，其行为类似于长度字段，但其数值使得不太可能解释长度。这暗示了校验和或其他“魔术”整数。
* “_可疑校验和块/Suspected checksummed block_” - 一个长数据块，其中任何更改总是触发相同的新执行路径。可能是由于在进行任何后续解析之前校验和或类似的完整性检查失败而引起的。
* “_魔法值部分/Magic value section_” - 一个通用token，其中更改会导致前面概述的二进制行为类型，但不符合任何其他标准。可能是一个原子级比较的关键字。
