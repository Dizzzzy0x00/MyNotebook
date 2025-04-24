---
cover: >-
  https://images.unsplash.com/photo-1726758004499-b4ea21408bcf?crop=entropy&cs=srgb&fm=jpg&ixid=M3wxOTcwMjR8MHwxfHJhbmRvbXx8fHx8fHx8fDE3Mjg0NDQ1MjN8&ixlib=rb-4.0.3&q=85
coverY: -140
---

# AI-Attack-NLP Attacks

## Part0 Start

一篇很好的入门文章，开启NLP attack的学习之路：

{% embed url="https://medium.com/besedo-engineering/nlp-attacks-part-1-why-you-shouldnt-trust-your-text-classification-models-37947853b1a5" %}

文章中将NLP Attacks归类为两类 :

* **Unicode Attacks (**_**“Bad” Characters**_**)**
  * 文章中对这种攻击的举例：“A particular example attracts your attention, a user called “**JohnWick546**” posted the following message: “You ІDІОТS !!”. You’re taken aback by this obvious display of hate, but you’re even more surprised that your text model classified it as non-toxic with a confidence score of _**0.70**_.  You quickly notice that the ІDІОТS word uses characters from both Latin and Cyrillic alphabets (_the Cyrillic alphabet contains some letters which visually look the same as Latin characters_), which explains why your model, despite “knowing” that the word “IDIOT” (_written with Latin characters_) is associated with toxicity, does not classify that comment as toxic, as the model has never seen such an elaborate character chain before, crafted to blind it."
  * unicode攻击的几种方式特点
    * The information contained in the text **stays the same**.
    * The characters contained in the text **stay visually similar** to the original text.
    * The targeted characters/tokens/words become **unrecognizable** by the model/filter.
  *   针对Unicode攻击的对抗（一些个人思考）：

      * 拼写错误检测和语义错误检测——>拼写错误纠正（后验概率，贝叶斯优化应用）
      * 文本顺序检测
      * 应用Unicode标准化（NFKD和/NFKC）
      * 应用OCR（光学字符识别），以避免不可见字符和同形符号的问题，因为OCR模型将以与人类相同的方式“看到”文本。 [**Bad Characters: Imperceptible NLP Attacks**](#user-content-fn-1)[^1][**https://zhuanlan.zhihu.com/p/675827506**](https://zhuanlan.zhihu.com/p/675827506)
      * 使用聚类方法将视觉上相似的Unicode字符组合在一起

      <figure><img src="../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure>
*   **Adversarial NLP Attacks** 对抗性NLP攻击

    * **“text containing&#x20;**_**“**_[_**perturbations**_](#user-content-fn-2)[^2]_**”**_**&#x20;to fool a machine learning model”**
    * 使用模型词汇表中没有的单词/标记（或由于其稀有性而表示不佳） -故意拼错单词 -罕见词汇 -混合语言（如果模型是用特定语言训练的） 交换特定单词/标记以混淆模型。
    * [**Towards a Robust Deep Neural Network in Texts: A Survey**](#user-content-fn-3)[^3]
    * 两种对抗性NLP攻击
      * **White-box attacks：**&#x653B;击者有模型架构和权重，（1）untargeted：让分类器分错即可，最大化损失函数以混淆模型，使其预测的类别与预期的类别不同（2）target：不仅要分类错误，还要让分类器给出指定类别
      * **Black-box attacks：**&#x653B;击者仅能访问模型API观察输入与输出的关系，不知道模型架构和权重手段：**word importance** metric 词重要性度量
        * 词重要性度量帮助攻击者识别在推理过程中对模型分类贡献最大的词。具体做法是通过多个**前向传递**（forward passes），逐个删除输入句子中的词或标记，观察模型在真值标签（ground truth label）上的置信度如何变化。
        * 如果删除某个词导致模型的置信度显著下降，说明该词在分类过程中起到了关键作用。反之，删除某个词对置信度影响不大，说明该词的重要性较低。
        * 在完成词重要性度量后，攻击者可以根据各个词的重要性进行排序，选择那些对分类最重要的词作为攻击目标
        * 用同义词替换这些最重要的词。通过使用预训练的掩码语言模型（如 BERT）或词嵌入（如 Word2Vec、GloVe），攻击者可以生成语义上相似的替换词，以最小化对句子整体语义的干扰
        * 在进行替换时，还需要考虑一些约束条件。例如，确保替换的词语保持**词性一致性（POS Consistency）**，也就是动词替换为动词，名词替换为名词，以保持句子的语法结构正确



    <figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Example of an adversarial NLP Attack [2]</p></figcaption></figure>



## Part1 使用openattack实现一次NLP attack

[^1]: **Bad Characters: Imperceptible NLP Attacks,** Nicholas Boucher et al. [https://arxiv.org/abs/2106.09898](https://arxiv.org/abs/2106.09898) (2021)

[^2]: 

[^3]: **Towards a Robust Deep Neural Network in Texts: A Survey,** Wenqi Wang et al. [https://arxiv.org/abs/1902.07285](https://arxiv.org/abs/1902.07285) (2019)
