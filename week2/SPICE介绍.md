# SPICE 介绍

> 论文： 《Spice: Semantic propositional image caption evaluation》

## 简介

SPICE在 ECCV2016 上提出，专门设计用来评价Image caption质量，在参考意义上比一般度量指标更高一些。

SPICE是 Semantic Propositional Image Caption Evaliation 的首字母缩写。我们之前介绍的几种方法，都会基于n-gram 来进行统计计算，但一个句子的好坏n-gram 只是其中一方面的体现，还有语义、结构等多种信息，SPICE的提出就是来解决这个问题的。

SPICE的基本思想是通过度量候选（candidate）和参考（reference）之间的相似度来做到两个句子语义上的匹配。首先利用semantic parser将句子都转换为场景图（scene graph），再通过场景图来得到以元组形式表现的信息，最终计算精确率、召回率，从而得到F1 分数。SPICE还有一个优点是不依赖于数据集的大小。

在具体介绍如何计算之前，我们先给出场景图的直观展示（图来自原论文）：

<img src="http://resource.mahc.host/img/image-20200710174143298.png" alt="图1：场景图与依赖树" style="zoom: 67%;" />



## 算法

### 符号定义

$$C$$ :物体的类别集合

$$R$$ :物体间的关系种类集合

$$A$$ : 物体的属性类别集合

$$c$$ : 一个caption

下面我们通过一个公式来表示将一个句子转换为场景图
$$
G(c) = \ <O(c),E(c),K(c)>
$$


上式中，$$O(c)$$ 表示句子 $$c$$ 中出现的物体，$$E(c) \subseteq O(c) \times R \times O(c) $$ 表示这些物体之间的关系，$$K(c) \subseteq O(c) \times A$$ 表示和这些物体相关联的属性。

再给出更具体的计算步骤前，我们再给出一个更完整的过程图帮助读者对这个方法有感性的认识（来自原文图2）：

<img src="http://resource.mahc.host/img/image-20200711144902313.png" alt="图2：图像与场景图" style="zoom: 67%;" />



上图中一张图像有5个人工标注的句子，右侧是是其中一个句子对应的场景图。

### 计算

1. 首先将一个句子转换为依赖树的形式（见图1顶部），这个过程依赖于一个分析工具PCFG（Stanford Scene Graph Parser的变种）,全名为Probabilistic Context-Free Grammar。（本文对其略有改进适配，详细见论文3.1节）

2. 把上面得到的依赖树通过9条简单的语法规则抽取出规范化的物体，关系以及属性，然后组成场景图。

3. 从场景图中以元组的形式抽取相关信息，用下面的公式来表示：
   $$
   T(G(c)) \triangleq O(c) \cup E(c) \cup K(c)
   $$
   每个元组内都会包含一个、两个或者三个元素，分别表示物体，关系，或者属性。下面给出图一中句子的抽取例子：
   $$
   \{(girl),(court),(girl,young),(girl,standing),(court,tennis),(girl,on-top-of,court)     \}
   $$
   
4. 计算

   定义一个操作符 $$\otimes$$  为两个场景图的元组的二项匹配，会返回匹配的元组，下面给出相关计算公式：

   * 精确率 
     $$
     P(c,S) = \frac {|T(G(c)) \otimes T(G(S))|}   {|T(G(c))|}
     $$

   * 召回率
     $$
     R(c,S) = \frac {|T(G(c)) \otimes T(G(S)) |}   {|T(G(S))|}
     $$

   * 定义 SPICE
     $$
     SPICE(c,S) = F_1 (c,S) = \frac {2\cdot P(c,S) \cdot R(c,S)}   {P(c,S)+ R(c,S)}
     $$



​	注意：元组的匹配不是单纯的字符串匹配，还会考虑词形词根的相似性，或者在wordnet 中的同义词。



## 参考资料

1. [一文全解Image Captioning中常用的自动度量](https://blog.csdn.net/luo3300612/article/details/100924577)
2. Anderson, P., et al. Spice: Semantic propositional image caption evaluation. ECCV. 2016. Springer.