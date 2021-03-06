# CIDEr

> 论文 《CIDEr: Consensus-based Image Description Evaluation》

## 简介

CIDEr ： Consensus-based Image Description Evaluation，直译为基于共识的图像描述评估

此评价标准是Vedantam 在2015年的CVPR（计算机视觉与模式识别大会）上提出来的，主要针对的问题是图像描述任务。

研究者认为过去的评价指标和人工评价有较强的相关性，但是无法统一到一个度量标准下来评价其与人的相似性，为了解决这个问题，提出了CIDEr。其优点还在于避免了因为一些常见词的出现而给句子打了高分，因此更加重视更有信息量的词。

CIDEr 的基本原理是把每个句子都看成是“文档”，然后引入TF-IDF（简要解释见下文）的方法来度量候选语句和参考语句之间的相似性，所以CIDEr可以看作是BLEU和向量空间模型的结合。最后研究者证明CIDEr与人工共识上的匹配度要好于其他评价指标。

## 算法

### TF-IDF

TF： term frequency

TF = 关键词出现次数 / 文章总词数 （注：一般会去除虚词等常见词 例如 “的”）

IDF： inverse document frequency

$$IDF = log (D/D_w)$$ 其中$$D$$表示文档的总个数，$$D_w$$ 表示包含词 w的文档的个数

IDF 的思想在于包含某词的文档的个数越多，则这个词越不重要。

TF-IDF：

$$相关性 = TF_1 * IDF_1 + ... + TF_N * IDF_N$$

### 余弦相似性

相信读者在高中的时候都已经学习过余弦值了，余弦值一个很重要的应用就是通过测量二者的夹角来衡量两个向量的相似度。如果二者的夹角是0°，则余弦值为1，表明两个向量的方向是完全一致的，其他角度同理。一般来说，余弦相似度用于正空间，也就是给出的值属于\[0,1\].

在这里我们需要注意的是，余弦相似性不仅仅用在二维空间中，高维空间也很常用，在后面的学习中，读者会经常用到这个方法去判断两个向量（本质上可能是文章、图像等）的相似性。

这里简单的给出两个向量的余弦值计算公式（欧几里得点积）：

$$
a · b = \| a\| \|b\| cos \theta
$$

则相似度定义如下：

$$
similarity = cos(\theta) = \frac {A·B}  {\|A\| \|B\|}
$$

### 计算

为了更清楚的描述CIDEr，我们首先给出数据集的相关符号表示：

#### 定义符号

$$I$$ : 图像集合

$$I_i$$ : 图像 i

$$C=\{c_1,c_2,...,c_N\}$$ ：表示待评测的句子，即候选集

$$c_i$$ : 图像i 的预测句子

$$S_i = \{s_{i1},s_{i2},...,s_{im}\}$$ : 表示第i张图像的m个参考句子，$$s_{ij}$$ 表示图像i 的第 j个参考句子

$$w_k$$ : n-gram 的其中一个，在本文中，n 可以是1-4，当n=1时，$$w_k$$ 表示 1-gram 集合的第k个单词

$$h_{k}(s_{ij})$$ : $$w_k$$ 在参考句子中出现的次数

$$h_k(c_i)$$ : $$w_k$$ 在候选句子中出现的次数

#### TF-IDF计算

针对某图像i 所对应的候选句子 $$c_i$$ 来计算TF-IDF\(论文中以计算$$s_{ij}$$ 为例\)，某n-gram $$w_k$$ 计算如下：

$$
TF(k) = \frac {h_k(c_i)}  {\sum h_l (c_i)}
$$

上式中，$$h_k(c_i)$$ 表示n-gram $$w_k$$ 在句子$$c_i$$ 中出现的次数，分母表示所有的n-grams（例如1-grams 中所有单词） $$w_l$$在句子 $$c_i$$中出现的次数之和。

$$
IDF(k) = log (\frac {|I|}   {\sum_{I_i \in I } min(1,h_k(c_i)) })
$$

上式中，分母表示n-gram $$w_k$$ 在数据集文本中总共出现的次数，最小值是1，最大值是N。

则TF-IDF的表示为：

$$
g_k(c_i) = TF(k) * IDF (k)
$$

#### CIDEr 计算

> 下面给出CIDEr计算解释很完美的一张图，更多信息见参考资料5

![CIDEr&#x8BA1;&#x7B97;&#x516C;&#x5F0F;](https://smailnjueducn-my.sharepoint.com/:i:/g/personal/mahc_smail_nju_edu_cn/EWJ64chWk-hGm2w5JKiFSLkBTOrrgY0YNYfB9tfPJBI7yw?e=mLgJ1B)

笔者注：$$m$$ 表示一个图像其参考句子（reference） 的个数；在本文中 $$N$$ = 4, $$w_n = \frac {1} {N}$$

#### CIDEr计算举例

> 为了使读者对上述计算更加清晰，这里我们选择一个参考句子，n=1 时的CIDEr分数计算（参考资料4）

候选集 ： $$c_1$$ = {我 吃 饭 了 吗}

参照集： $$S_{11}$$ = {他 早 上 吃 饭 了}

1. 把相关单词归结到词根等，例如 fishes 记录为 fish
2. 结合候选集和参照集，形成 1-gram 集合 {我 吃 饭 了 吗 他 早 上}
3. 分别计算 $$w_1,w_2,w_3,...,w_8$$ 的 TF-IDF \(k=1 时代表 ’我‘ \)

   $$g_1(c_1) = 1/5 * log(1/1) = 0$$ **注意：一般数据集为1时，为了防止0的出现，设定IDF= 1 ，因此** $$g_1(c_1) = 0.2$$ ****

   $$g_1(s_11) = 0$$

   $$g_2(c_1) = 0.2$$

   $$g_2(s_{11}) = 0.2$$

   以此类推，我们可以得到n=1 ，m=1 ，数据集大小为1 时的计算结果：

   | 1-gram $$w_k$$ | $$g_k(c_1)$$ | $$g_k(s_{11})$$ |
   | :--- | :--- | :--- |
   | 我 | 0.2 | 0 |
   | 吃 | 0.2 | 0.2 |
   | 饭 | 0.2 | 0.2 |
   | 了 | 0.2 | 0.2 |
   | 吗 | 0.2 | 0 |
   | 他 | 0 | 0.2 |
   | 早 | 0 | 0.2 |
   | 上 | 0 | 0.2 |

4. 计算$$CIDEr_1$$

   $$
   CIDEr_1 = 1 * \frac {[0.2 \ 0.2 \ 0.2 \ 0.2 \ 0.2 \ 0 \ 0 \ 0 ]^T * [0\ 0.2 \ 0.2\ 0.2\ 0\ 0.2\ 0.2\ 0.2 ]}  {\sqrt{0.2^2 + 0.2^2 +0.2^2 +0.2^2 +0.2^2} + \sqrt{0.2^2 +0.2^2 +0.2^2 +0.2^2 +0.2^2 +0.2^2}} = 0.5477
   $$

5. 计算 CIDEr

   $$
   CIDEr =  \frac{1}{1} * CIDEr_1 = 0.5477
   $$

#### CIDEr-D

在这里我们介绍一个词“gaming”，指代一个指标的潜在漏洞，例如一个参考句子在机器评测下的指标很高，但是人工评价很差，这就是这个度量的一个gaming。论文作者在最后就提出了CIDEr的两个缺点：

* CIDEr取词根的操作会让一些动词的原型和名词匹配成功
* 高confidence的词重复出现的长句的CIDEr得分也很高

为了解决上述的两个缺点，作者提出了CIDEr-D

CIDEr-D采用以下方法来解决：

* 对于动词原形和名词匹配成功的问题，CIDEr-D不再取词根
* 对于包括高confidence的词的长句，CIDEr-D引入了gaussian penlty来惩罚candidate和reference的句长差别，并且通过对n-gram计数的clip操作不再计算candidate中出现次数超过reference的n-gram。

下面给出CIDEr-D的计算公式\(来源见原论文\)：

$$
CIDEr-D_n(c_i,S_i) = \frac {10} {m} \sum_{j} e^{\frac {-(l(c_i)-l(s_{ij}))^2}  {2\sigma^2}} * \frac {min(g^n(c_i),g^n(s_{ij}))*g^n(s_{ij})}  {\|g^n(c^i)\| \|g^n(s^{ij})\|}
$$

上式中$$l$$ 代表对应句子的长度，原论文中 $$\sigma = 6$$ ，因子10 只是为了让最后的分数和其他评价分数在数值上类似。

$$
CIDEr-D(c_i,S_i) = \sum_{n=1} ^{N} w_n CIDEr-D_n (c_i,S_i)
$$

当然CIDEr-D也存在一些缺点，例如SPICE 论文中指出CIDEr因为引入TF-IDF，所以指标分数受到整个数据集规模的影响。

## 参考资料

1. [浅述自然语言处理机器翻译常用评价度量](https://blog.csdn.net/joshuaxx316/article/details/58696552)
2. [文本挖掘预处理之TF-IDF](https://www.cnblogs.com/pinard/p/6693230.html)
3. [余弦相似性](https://zh.wikipedia.org/wiki/%E4%BD%99%E5%BC%A6%E7%9B%B8%E4%BC%BC%E6%80%A7)
4. [CIDEr的计算](https://blog.csdn.net/wl1710582732/article/details/84202254)
5. [CIDEr海报](https://filebox.ece.vt.edu/~vrama91/CIDEr_miscellanous/ciderPosterCVPR15.pdf)
6. [一文全解Image Captioning中常用的自动度量](https://blog.csdn.net/luo3300612/article/details/100924577)

