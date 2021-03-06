# 指标评价思考

> 近年来（尤指最近两年）基于深度学习来解决图像描述问题的论文越来越多，而如何评价一个模型的好坏变得至关重要。本节内容即针对目前的指标还有多大的参考价值这一问题作简要实验。

## 评价指标介绍

​ 图像描述的的质量好坏主要从三个方面来评价：

> * 是否正确表述图像的内容句子
> * 是否符合人类的阅读习惯
> * 句子是否具有多样性，例如不同的视角、不同层次等

​ 一个句子质量究竟如何，最好的评价方法就是请专家评测，这样作的好处在于准确、灵活，缺点是效率低，代价高，还有就是不同的研究团队没有统一的评价指标。还有另一种选择就是机器评测，目前论文间模型质量评测主要就是利用机器按照一定的规则进行评测。

​ 目前主要使用的评测指标简要介绍如下，**更详细的相关说明见本周提供的其他相关文件**：

表2： 常用评价指标简介

| 指标名称 | 指标简介 |
| :--- | :--- |
| $$BLEU^{[1]}$$ | 双语互译质量辅助工具，句子级别的判断。基于精确度（分子是共现次数，分母是人工标注的句子中n-gram数量）的相似性度量方法。多用于机器翻译领域 |
| $$ROUGE^{[2]}$$ | 基于召回率的相似性度量方法，和BLEU很类似，但分母是生成句子的n-gram数量。多用于神经网络机器翻译和自动文本摘要质量测评 |
| $$METEOR^{[3]}$$ | 上述评测方法（BLEU/ROUGE）只是单纯的匹配计算，METEOR中引入WordNet词典，实现同义词匹配等功能。最后的评价指标可以认为是上述召回率和精确率的调和平均。因为引入了同义词判别，所以结果上和人工评价更有相关性。但缺点是其中有些超参需要调节 |
| $$CIDEr^{[4]}$$ | 2015年计算机视觉大会提出，专门用于图像标注问题。目的是为了评价生成的语言和人类写的之间的相似度。计算n元语言模型在参考描述句子和模型生成待评测句子的共现概率。研究证明其在与人共识的匹配上好于其他测评 |
| $$SPICE^{[5]}$$ | ECCV2016年提出，专门设计用来评价Image caption质量。先把生成/参考的描述句子都转换为基于图的语义表示，即场景图，然后进行对比计算。 |

## 实验设计与结果分析

### 设计与实验

​ 评价指标现在还有多大的意义，是不是意味着指标越大，句子质量越好，越接近人类的描述？我设计了一个简单的实验，即利用上述五种评测方法来评价人工标注的句子的质量，以此来说明指标在评价人工标注的数据上限是多少，指标越高是不是说明句子距离人工标注越接近。

​ MS COCO2014 数据集，我们依旧采用Karpathy分割办法，在5000个样本的测试集中，每张图像有5个标注的句子，记作A-E。我们先将A视作生成的句子，B-E视为参考句子来计算上述5个指标点。然后依次使用B-E作为生成的句子，其余作为参考句子来计算评价指标。最后取5次结果平均来作为人工标注的评测指标。最终评测结果见表3.

表3 ： 人工标注评价结果与相关模型结果对比

| 模型 | Bleu\_1 | Bleu\_2 | Bleu\_3 | Bleu\_4 | METEOR | ROUGE\_L | CIDEr | SPICE |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| $$SoftAtt^{[6]}$$ | 70.7 | 49.2 | 34.4 | 24.3 | 23.90 | — | — | — |
| $$NIC^{[7]}$$ | — | — | — | 27.7 | 23.7 | — | 85.5 | — |
| $$BUTD^{[8]}$$ | 79.8 | — | — | 36.3 | 27.7 | 56.9 | 120.1 | 21.4 |
| $$SCST^{[9]}$$ | — | — | — | 34.2 | 26.7 | 55.7 | 114.0 | — |
| $$NBT^{[10]}$$ | 75.5 | — | — | 34.7 | 27.1 | — | 107.2 | 20.1 |
| $$AoA^{[11]}$$ | 80.2 | — | — | 38.9 | 29.2 | 58.8 | 129.8 | 22.4 |
| **Human** | **62.98** | **43.38** | **29.1** | **19.42** | **24.16** | **46.68** | **87.74** | **21.12** |

### 分析

​ 通过表3我将人工标注的评测指标和近年来比较有代表性的工作指标进行了对比，可以看到人工标注的评测指标几乎是最低的。这个结果是令人难以置信的，为了确保实验数据的准确性，对比了论文\[7\]中给出的人工标注自动评测结果，发现指标接近，基本可以说明我计算的结果有很强的可信度。

​ 通过上述对比，可以得出以下结论：

> * 指标点高并不意味着生成的句子质量一定更加的接近人工评价
> * 指标点越来越高并不意味着句子质量一定越来越好
> * 各项评价指标并不能很好的对人工标注进行评价，指导生成的句子可能质量更差

## 下一阶段工作

​ 为了更进一步的验证上述结论，并得出定量分析，以此来作为对评价指标的具体认识。我们计划接下来两周进行人工评价来判断当前各个模型生成的句子的质量如何。具体步骤如下：

1. 简单了解上述及其他经典模型
2. 对各个模型的生成结果进行随机采样，掩盖模型信息
3. 设计人工评价给分点，人工对各个模型生成的结果进行盲评
4. 对得出的数据结果进行分析

​ 经了解，包括图像描述在内的机器翻译、自动摘要等多个方向都有评价指标难以评估模型的问题，笔者认为，基于数据驱动的深度学习模型要想更进一步的有所突破，必须直面现有评价指标无法跟上模型快速发展的问题。

​ 本次暑期夏令营的后十天，将与大家一起领略不同模型的生成质量，了解当前深度学习背景下的语言模型质量，从图像描述一窥机器学习的现状，进而认识机器学习方法下的人工智能进展，最终以小组（分组）形式提交两份书面报告。

## 参考资料

1. Papineni, K., et al. BLEU: a method for automatic evaluation of machine translation. in Proceedings of the 40th annual meeting on association for computational linguistics. 2002. Association for Computational Linguistics.
2. Lin, C.-Y. ROUGE: A Package for Automatic Evaluation of Summaries. in Text Summarization Branches Out. 2004. Barcelona, Spain: Association for Computational Linguistics.
3. Banerjee, S. and A. Lavie. METEOR: An automatic metric for MT evaluation with improved correlation with human judgments. in Proceedings of the acl workshop on intrinsic and extrinsic evaluation measures for machine translation and/or summarization. 2005.
4. Vedantam, R., C.L. Zitnick, and D. Parikh, CIDEr: Consensus-based Image Description Evaluation. 2015.
5. Anderson, P., et al. Spice: Semantic propositional image caption evaluation. in European Conference on Computer Vision. 2016. Springer.
6. Xu, K., et al. Show, attend and tell: Neural image caption generation with visual attention. in International conference on machine learning. 2015.
7. Vinyals, O., et al. Show and tell: A neural image caption generator. in Proceedings of the IEEE conference on computer vision and pattern recognition. 2015.
8. Anderson, P., et al., Bottom-up and top-down attention for image captioning and visual question answering. Proceedings of the IEEE conference on computer vision and pattern recognition, 2018: p. 6077-6086.
9. Rennie, S.J., et al. Self-critical sequence training for image captioning. in Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition. 2017.
10. Lu, J., et al. Neural baby talk. in Proceedings of the IEEE conference on computer vision and pattern recognition. 2018.
11. Huang, L., et al. Attention on attention for image captioning. in Proceedings of the IEEE International Conference on Computer Vision. 2019.

