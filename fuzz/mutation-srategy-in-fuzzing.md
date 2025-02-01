---
description: 模糊测试的变异策略
---

# Mutation Srategy in Fuzzing

根据模糊测试工具生成测试用例的不同，目前分为两类：**基于生成和基于变异**。

* 基于生成的策略：在这种策略中，通常根据已知的协议或接口规范，通过建模，对输入数据进行生成，以此生成测试用例。这种策略的优点是可以精确地控制生成的数据，并制定更加合理或者复杂的测试用例。然而，对于复杂的协议或接口，需要投入大量的时间和人力进行分析建模，这极大地增加了测试的复杂性和难度。
* 基于变异的策略：该策略的基本思想是对已知的有效输入数据或数据样本进行适当的变异或修改，生成新的测试用例。例如，通过改变或交换数据样本中的字节，插入或删除某些字节，或者以其他方式修改数据样本。这种策略的优点是操作简洁直接，通过随机或有目的的变异，可以生成大量新的测试用例。但是，它也存在一定的难点——如何设计变异策略，一旦变异策略设计不周，可能导致生成的测试用例覆盖不全，无法达到理想的测试深度。因此，找到合适的变异策略和变异方法对基于变异的模糊测试来说，具有至关重要的作用。

变异策略是模糊测试器的精髓所在。在基于变异的模糊测试环节中，变异过程是输入构造步骤的重要组成部分。一旦选定并筛选了合适的种子，需要依赖有效的变异策略来确定适宜的变异位置，再调用不同的变异方法来产生众多的输入文件。在这个关键过程中，变异策略和变异方法的选择直接影响到生成的输入文件的质量。

如果生成的变异输入与原始的种子文件相差无几，那么很可能无法有效地提升测试的覆盖率。相对地，若是变异过程给种子文件带来了过于剧烈的变化，可能会破坏文件的语法结构，导致生成大量无效的输入文件，从而降低模糊测试的效率。可见，设计变异策略需要权衡种子文件的变异程度与保持其有效性之间的关系，以提高模糊测试的整体性能。

模糊测试器的变异策略是用于决定种子文件中的什么位置进行何种变异，按照其实现思路可以将变异策略分为两类：非导向型变异策略和导向型变异策略。

顾名思义，非导向型变异策略随机选取变异位置，这样可以快速生成大量的输入文件。与非导向型变异策略不同，导向型变异策略根据预处理阶段或模糊测试循环过程中得到的信息有目性地选择变异位置，使用适当的变异方法对种子文件进行变异，尽可能使种子文件经过变异可以提升代码覆盖范围。

调研了目前相关变异策略的研究，汇总其变异策略类型和详细使用的技术如下表：

<table data-header-hidden><thead><tr><th width="135">Fuzzer</th><th width="93">变异策略类型</th><th>详细变异策略</th></tr></thead><tbody><tr><td>AFL[1]</td><td>非导向型</td><td>确定性变异阶段（如bit flip、arithmetic、interest等策略）和随机变异阶段（havoc、splice）</td></tr><tr><td>AFL++[7]</td><td>非导向型</td><td>相较于AFL扩展了更多的变异策略，此外AFL++支持了自定义的变异，允许相关研究在 AFL++ 之上构建新的变异策略</td></tr><tr><td>Driller[8]</td><td>导向型</td><td>在AFL的基础上加入了动态符号执行引擎，定义了复杂检查即程序中那些条件过于苛刻而无法被随机变异满足的条件语句。当模糊器在循环过程中遇到复杂检查被卡住时，调用动态符号执行模块寻找能够到达新路径的输入，从而快速满足复杂检查的条件。</td></tr><tr><td>DigFuzz[9]</td><td>导向型</td><td>类似Driller，也是在模糊测试中引入符号执行，优化了传统符号执行的代价，用一种代价较小的方法评估每条路径的难度，按照探索难度对路径进行排序，将难度最大的路径留给符号执行进行探索</td></tr><tr><td>VUzzer[10]</td><td>导向型</td><td>使用了静态分析和动态污点分析的方法，其中污点分析阶段中得到的信息会反馈给输入选择阶段，分析追踪MAGIC比较分支中的常量字节，可以定位其输入偏移位置，能指导模糊测试快速地覆盖这些MAGIC比较分支，以改善输入选择和突变策略。</td></tr><tr><td>Angora[11]</td><td>导向型</td><td>使用字节级别的污点分析技术来搜索种子中能够提高代码覆盖率的变异操作和变异位置, 并在模糊测试过程中通过梯度下降算法解决路径约束问题</td></tr><tr><td>TaintScope[12]</td><td>导向型</td><td>利用动态污点分析技术定位校验和检查分支，有效地辅助模糊测试绕过校验和检查分支</td></tr><tr><td>TIFF[13]</td><td>导向型</td><td>基于类型推断的模糊框架，使用基本类型（如32 位整数、字符串、数组等）标记输入的每个偏移量，同时导出特定类别错误的突变规则，结合内存数据推断类型、结构识别和动态污点分技术的变异策略</td></tr><tr><td>GreyOne[14]</td><td>导向型</td><td>引入污点推理方法FTI，采用最直观的推断方式，对种子中某个字节变异后进行测试，如果发现种子的路径上的分支变量值发生变化，那么受影响的变量数据依赖于该变异字节。</td></tr><tr><td>PATA[15]</td><td>导向型</td><td>利用路径感知污点分析的结果来指导fuzzer的突变过程，在推理过程中, 模糊器对输入的每个字节进行扰动, 并监视变量出现的值变化, 以识别关键字节</td></tr><tr><td><p><a href="https://www.zhangqiaokeyan.com/academic-conference-foreign_meeting_thesis/0705016453811.html">Deep Reinforcement</a></p><p>[16]</p></td><td>导向型</td><td>使用马尔可夫决策过程的概念将Fuzz形式化为强化学习问题，学习种子文件与覆盖率之间的关系，得到能够增加代码覆盖的变异操作及变异位置</td></tr><tr><td>Neuzz[17]</td><td>导向型</td><td>使用一个基于RNN（循环神经网络）的神经网络来对程序进行学习和平滑处理。通过将输入样本和对应的代码路径作为训练数据，NEUZZ学习到输入样本在不同代码路径上的执行特征，使用模型计算的梯度指导变异策略</td></tr><tr><td>MTFuzz[18]</td><td>导向型</td><td>使用多任务神经网络（multi-task neural network）来解决目标程序输入空间高维且稀疏导致的难题，基于多个相关任务的不同训练样本，实现紧凑的嵌入，将大部分突变集中在梯度高的嵌入部分来指导突变过程</td></tr></tbody></table>

接下来对上面的变异策略进行归类和讨论。

### 非导向型策略

以AFL为代表的模糊测试器使用非导向型策略，这种简单的非导向型变异策略带有较大的随机性，可能对种子的语法结构产生破坏性影响，使得经过变异的输入文件难以通过待检测程序的语法审查。在处理如今巨大而复杂的程序时，这种策略相较于其他导向型策略效率较为低下。

### 混合模糊测试

混合模糊测试（Hybrid Fuzzing），是将灰盒模糊测试与符号执行相结合的漏洞检测方法。由于2.1中提到的传统的非导向型模糊测试手法往往难以生成有效的输入样本来触发复杂的程序路径分支，混合模糊测试引入符号执行，采集与执行路径相关的指令，并将其转化为可解的条件约束，面对复杂的路径分支，符号执行能够获得触发此路径的所有条件约束，并通过约束求解方法获取有效的输入以触发该路径。混合模糊测试便是在同一被测程序中同时应用模糊测试和符号执行的方法。

而符号执行技术又可以分为静态符号执行和动态符号执行两种。静态符号执行常常由于程序中的循环和递归结构而导致路径爆炸，同时还可能由于路径约束中包含诸如获取哈希值等操作，使得约束求解失败。

动态符号执行通过结合实际执行与符号化执行来跟踪程序的实际和符号化状态。这种方法通过使用实际值替换难以求解的约束，可以缓解静态符号执行所面临的问题，并采用深度优先的搜索策略对目标程序进行探索。但是使用动态符号执行技术的一个问题是，由于程序分支的存在，在大型复杂的程序上容易出现路径爆炸的问题，解决的一种办法是通过启发式的方法,选择比较重要的路径进行探索。如图展示了一个使用符号执行来指导模糊测试变异策略的基本流程。

<figure><img src="../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption><p>符号执行指导的变异策略基本流程</p></figcaption></figure>

### 数据流敏感的模糊测试

使用数据流分析技术(污点分析)来指导模糊测试覆盖复杂约束，这种技术会监测程序中受到预设污染源（如输入）污染的数据，其目的在于追踪污染源与收集点（如含有敏感信息的程序数据）之间的信息流。

污点分析同样可以分为静态污点分析和动态污点分析。静态污点分析通过静态地对程序进行分析，获取程序的控制流图、抽象语法树等信息，并基于数据流以及依赖关系进行污点分析。而动态污点分析则在程序真实运行的过程中，利用程序的动态执行信息来进行污点分析。

将污点分析技术应用到模糊测试中,并降低其资源消耗是近年来的重要研究方向之一，比如GreyOne\[14]尝试通过减少污点分析跟踪的对象、降低污点分析的开销、提升模糊测试的检测效率,其设计思路如下图所示。

<figure><img src="../.gitbook/assets/image (6) (1).png" alt=""><figcaption><p>GreyOne污点分析技术指导的模糊测试设计流程</p></figcaption></figure>

### 深度学习指导的模糊测试

面对越来越复杂的程序，符号执行、污点分析技术应用到模糊测试中虽然有其优点，但因其路径爆炸、环境模拟不完整、符号建模占用大量内存等问题，不能扩展到大规模的程序中。如Neuzz等模糊器引入了神经网络，使用深度学习模型来学习到输入样本在不同代码路径上的执行特征，使用模型计算的梯度指导变异策略，详细内容可以参考我的另一篇文章

{% content-ref url="ai-with-fuzzing.md" %}
[ai-with-fuzzing.md](ai-with-fuzzing.md)
{% endcontent-ref %}

## 参考文章

\[1] Michal Zalewski. 2019. American Fuzzy Lop.\[OL] http://lcamtuf.coredump.cx/afl/ technical\_details.txt.

\[2] Bhme M , Pham V T , Roychoudhury A .Coverage-based Greybox Fuzzing as Markov Chain\[J].IEEE Transactions on Software Engineering, 2016.DOI:10.1145/2976749.2978428.

\[3] Gan S , Zhang C , Qin X ,et al.CollAFL: Path Sensitive Fuzzing\[C]//2018 IEEE Symposium on Security and Privacy (SP).IEEE Computer Society, 2018.DOI:10.1109/SP.2018.00040.

\[4] Dinesh S , Burow N , Xu D ,et al. RetroWrite: Statically Instrumenting COTS Binaries for Fuzzing and Sanitization\[C]//2020 IEEE Symposium on Security and Privacy (SP).IEEE, 2020.DOI:10.1109/SP40000.2020.00009.

\[5] Chen P , Chen H .Angora: Efficient Fuzzing by Principled Search\[C]//2018 IEEE Symposium on Security and Privacy (SP).IEEE, 2018.DOI:10.1109/SP.2018.00046.

\[6] Lyu C , Ji S , Zhang C ,et al. MOPT: optimized mutation scheduling for fuzzers\[C]//USENIX Security Symposium.2019.

\[7] Fioraldi A , Maier D , Eifeldt H ,et al.AFL++ : Combining Incremental Steps of Fuzzing Research.\[C]//USENIX Security Symposium.2020.

\[8] Stephens N, Grosen J, Salls C, et al. Driller: Augmenting fuzzing through selective symbolic execution\[C]//NDSS. 2016, 16(2016): 1-16.

\[9] Zhao L , Cao P , Duan Y ,et al.Probabilistic Path Prioritization for Hybrid Fuzzing\[J].IEEE Transactions on Dependable and Secure Computing, 2020, PP(99):1-1.DOI:10.1109/TDSC.2020.3042259.

\[10] Sanjay Rawat∗†,Vivek Jain‡,Ashish Kumar‡,et al.VUzzer: Application-aware Evolutionary Fuzzing\[C]//Ndss.2017.DOI:10.14722/ndss.2017.23404.

\[11] Chen P, Chen H. Angora: Efficient fuzzing by principled search\[C]//2018 IEEE Symposium on Security and Privacy (SP). IEEE, 2018: 711-725.

\[12] Wang T , Wei T , Gu G ,et al.TaintScope: A Checksum-Aware Directed Fuzzing Tool for Automatic Software Vulnerability Detection\[J].IEEE, 2010.DOI:10.1109/SP.2010.37.

\[13] Jain V , Rawat S , Giuffrida C ,et al.TIFF: Using Input Type Inference To Improve Fuzzing\[C]//the 34th Annual Computer Security Applications Conference.2018.DOI:10.1145/3274694.3274746.

\[14] Gan S , Zhang C , Chen P ,et al.{GREYONE: Data Flow Sensitive Fuzzing\[C]//USENIX Security Symposium.2020.

\[15] LIANG J,WANG M, ZHOU C,et al.PATA: Fuzzing with Path Aware Taint Analysis\[C/OL]//2022 IEEE Symposium on Security and Privacy (SP). 2022: 1-17\[2024-05-20].&#x20;

\[16] Bottinger K , Godefroid P , Singh R .Deep Reinforcement Fuzzing\[C]//2018 IEEE Security and Privacy Workshops (SPW).IEEE, 2018.DOI:10.1109/SPW.2018.00026.

\[17] She D , Pei K , Epstein D ,et al.NEUZZ: Efficient Fuzzing with Neural Program Smoothing\[J]. 2018.DOI:10.48550/arXiv.1807.05620.

\[18] She D , Krishna R , Yan L ,et al.MTFuzz: Fuzzing with a Multi-Task Neural Network\[J].arXiv e-prints, 2020.DOI:10.1145/3368089.3409723.

