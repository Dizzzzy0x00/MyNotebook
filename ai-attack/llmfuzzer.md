# LLMFuzzer

## JailFuzzer：由LLM-Agent驱动的自动化文生图模型越狱框架，CCF-A <a href="#activity-name" id="activity-name"></a>

{% embed url="https://ieeexplore.ieee.org/abstract/document/11023298" %}

源码：

{% embed url="https://github.com/YingkaiD/JailFuzzer" %}

### 背景

“越狱攻击”（Jailbreaking Attack）——即通过精心设计的提示词（Prompt）绕过模型的安全机制，生成不安全内容（NSFW）——是一个严峻的挑战。现有的越狱方法大多存在访问要求不切实际（如需要白盒访问）、生成的提示词不自然易被检测、搜索空间受限或查询成本高昂等局限。

一个由大型语言模型（LLM）智能体（Agent）驱动的新型模糊测试（Fuzz-Testing）框架。它能在黑盒设置下，高效地生成自然且语义完整的越狱提示词。JailFuzzer 创新地将模糊测试的经典三要素——种子池、引导诱变引擎和预言机（Oracle）——与强大的 LLM-Agent 相结合，显著提升了攻击的效率和隐蔽性。实验结果表明，JailFuzzer 在越狱 T2I 模型方面表现卓越：它生成的提示词自然流畅，难以被传统防御机制发现；同时，它以极低的查询开销实现了极高的越狱成功率（对多数安全过滤器接近100%，平均仅需 4.6 次查询），全面超越了现有方法。这项研究揭示了当前AIGC模型安全机制的脆弱性，并为未来构建更强大的防御体系奠定了基础。

### 设计与实现

JailFuzzer 的核心思想是将经典的模糊测试框架与先进的 LLM-Agent 相结合。其总体框架图如下所示，主要由种子池（Seed Pool）、引导诱变引擎（Guided Mutation Engine）和预言机（Oracle Function）三部分构成。整个攻击流程被设计成一个多轮攻击循环，灵感来源于“指数退避”策略，使得框架能从过去的成败经验中学习，动态进化。

<figure><img src="../.gitbook/assets/302cba4a9eac45c03758ece1f7a2ee9c.png" alt=""><figcaption></figcaption></figure>

### TurboFuzzLLM <a href="#title" id="title"></a>

{% embed url="https://aclanthology.org/2025.naacl-industry.43/" %}
