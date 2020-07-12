# METEOR 介绍

> 提出论文 《METEOR: An automatic metric for MT evaluation with improved correlation with human judgments》



## 简介

METEOR ： Metric for Evaluation of Translation with Explicit ORdering，直译为 显式排序的翻译评估指标

METEOR 由Banerjee和Lavie 在2004年提出，该度量方法参考了召回率的意义（关于召回率参考BLEU章节介绍）。细心的读者可能会发现，METEOR和BLEU在提出时间上相差不大，实际上METEOR也是为了克服BLEU的一些缺点而提出来的。BLEU是基于精度的标准，其研究表明召回率的标准和单纯基于精度的标准分别和人工判断的结果相比，前者具有更高的相关性。实验表明METEOR和人类评价的在文档级别的相关性为0.964，而BLEU是0.817，在句子级别上，METEOR的最大相关性为0.403.

具体来讲，METEOR是基于单字的精确率和召回率的调和平均，但召回率的权重比精确率的更高一些。当然，它还有一些在其他方法中没有发现的功能，例如词干和同义词的匹配。



## 算法

### 建立映射

METEOR的基本评价单位是句子，首先算法会建立两个句子（标注和预测）之间的对齐关系（单词间），用下面的图来描述可能会更容易接受一些：

<p>对齐关系(左右两类对齐)：</p>
<center>
<img src="http://resource.mahc.host/img/METEOR-alignment-a.png" style="zoom:50%;"/>
<img src="http://resource.mahc.host/img/METEOR-alignment-b.png" style="zoom:50%;" />
</center>



关于对齐的限制条件如下：

* 每个候选句子（candidate）中的单词必须对应参考句子（reference）中的0个和1个句子

那么有读者会问，如果两个句子之间存在多种对应关系呢（映射数量相同，如上图）？在这里，会选择交叉线条较少的映射。例如上面的例子中，我们选择左边的映射关系。



### 计算

1. 精确率
   $$
   P = \frac {m} { w_t}
   $$
   $$m$$ 是候选句子和参考句子中共同出现的单词个数， $$w_t$$  是候选句子中的单词个数，这是标准的精确率的定义，无需多言。

2. 召回率
   $$
   R = \frac {m}  {w_r}
   $$
   $$m$$ 还是候选句子和参考句子中共同出现的单词个数，$$w_r$$ 是 参考句子中的单词个数

3. 调和平均
   $$
   F_{mean} = \frac{PR}  {\alpha P + (1-\alpha) R}
   $$
   上面的参数$$\alpha$$ 就是用来调整精确率和召回率的参数

4. 匹配长度的惩罚

   以上我们只是考虑到单个词的匹配关系，那么连续几个单词匹配的话理应得到更高的分数，所有在这里有个惩罚因子，定义如下：
   $$
   pen = \gamma(\frac{ch} {m})^\theta
   $$
   上面的式子中，$$\gamma$$ 和 $$\theta$$  都是人为设定的参数，一般分别设置为0.5 和 3.另外，基于WordNet的同义词库， $$ch$$ 表示连续且有序的块的个数（这个值越大表明连续匹配的块越少，质量越差，最坏的情况是和m值相同，即没有两个以上连续词的匹配）， $$m$$ 表示候选句子中已经建立映射关系的单词（1-gram）的个数。  

5. 计算METEOR

   通过上述定义，下面正式给出METEOR的定义：
   $$
   METEOR = (1-pen) F_{mean}
   $$



注： 如果有多个参考答案，我们的策略是让候选句子分别和这些参考答案计算METEOR分数，最后选择分数最高的即可。



## 举例

> 未防止读者泄于阅读参考资料，本部分引用维基百科的例子部分,其中认为参数已经进行了设定

* Reference：	the	cat	sat	on	the	mat

* Hypothesis：	on	the	mat	sat	the	cat

  ```
  METEOR: 0.5000 = F_mean: 1.0000 × (1 − Penalty: 0.5000)    式（5）
  F_mean: 1.0000 = 10 × Precision: 1.0000 × Recall: 1.0000 / （Recall: 1.0000 + 9 × Precision: 1.0000）   式（3）
  Penalty: 0.5000 = 0.5 × (Fragmentation: 1.0000 ^3)  式（4）
  Fragmentation: 1.0000 = Chunks: 6.0000 / Matches: 6.0000  式（4）括号内
  ```

* Reference：	the	cat	sat	on	the	mat

* Hypothesis：	the	cat	sat	on	the	mat

  ```
  Score: 0.9977 = Fmean: 1.0000 × (1 − Penalty: 0.0023)
  Fmean: 1.0000 = 10 × Precision: 1.0000 × Recall: 1.0000 / （Recall: 1.0000 + 9 × Precision: 1.0000）
  Penalty: 0.0023 = 0.5 × (Fragmentation: 0.1667 ^3) 
  Fragmentation: 0.1667 = Chunks: 1.0000 / Matches: 6.0000
  ```

* Reference:	the	cat		        sat	on	the	mat

* Hypothesis:	the	cat	was	sat	on	the	mat

  ```
  Score: 0.9654 = Fmean: 0.9836 × (1 − Penalty: 0.0185)
  Fmean: 0.9836 = 10 × Precision: 0.8571 × Recall: 1.0000 / （Recall: 1.0000 + 9 × Precision: 0.8571）
  Penalty: 0.0185 = 0.5 × (Fragmentation: 0.3333 ^3)
  Fragmentation: 0.3333 = Chunks: 2.0000 / Matches: 6.0000
  ```

  


## 参考资料

1. [浅述自然语言处理机器翻译常用评价度量](https://blog.csdn.net/joshuaxx316/article/details/58696552)
2. wiki：[METEOR](https://en.wikipedia.org/wiki/METEOR)
3. [文本生成评价方法](https://zhuanlan.zhihu.com/p/108630305)

