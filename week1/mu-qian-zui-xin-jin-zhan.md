# 目前最新进展

> 最近这两年，Image Caption在AAAI、CVPR等顶级会议上发表了数篇论文，这些文章一般从模型改进、新应用、新指标等几个角度来阐述。本节简要介绍相关工作的代表性文章。

## 最新具有代表性的论文

1. 《Unified vision-language pre-training for Image Captioning and VQA》 AAAI 2020.

   ​ 文章的“创新点”在于讲语言的预训练模型引入到图像描述和视觉问答中，并且这两个任务用的是同一个模型，然后在下游任务上进行fine-tune，结果显示，并且在两个任务上都达到了最好的结果，而预训练模型正是采用火热的Transformer技术。

2. 《Hierarchy Parsing for Image Captioning》 ICCV 2019.

   ​ 文章的创新点在于提出了分层次的对图像进行解析。在传统做法中，直接使用CNN网络对图像的特征进行抽取，在这篇文章中，作者对图像按照目标检测的手段进行分割，然后再分割基础上使用语义分割，然后把两次分割的结果以及原图构成一个树结构，由此得到了一个分层次的表示，最后通过解码器来得到结果。

3. 《Show, Recall, and Tell: Image Captioning with Recall Mechanism》 AAAI 2020.

   ​ 本文大胆的引入了传统的基于检索的方案。即首先在给定的数据库中找到和给定图片相关的词语作为备选。然后设计模型，在进行生成的过程中把结果分为两部分，一部分来源于拷贝，另一部分来源于生成。这和文本摘要领域中的拷贝机制有相似之处。

4. 《Image Captioning with Very Scarce Supervised Data: Adversarial Semi-Supervised Learning Approach》 EMNLP 2019.

   ​ 众所周知，深度学习过程中，标注数据是无论如何也绕不过去的。这篇论文即尝试使用较少的标注语料，通过对抗学习的方法，来生成有噪音的标注数据，从而解决图像描述任务。最后的结果显示较差，但这是一个很有意义的尝试。

5. 《Learning Long- and Short-Term User Literal-Preference with Multimodal Hierarchical Transformer Network for Personalized Image Caption》 AAAI 2020.

   ​ 这篇论文是对图像描述任务的拓展，十分有趣。具体来说，就是传统的图像描述结果使人感到十分的厌烦，结构僵化。本文通过Instagram等平台的用户历史数据进行建模，最后可以实现根据用户的照片来预测用户会发什么话，相当的有趣。这类创新可以划归到使生成的标题更加个性化这个范畴。

### 参考文献

1. Zhou, Luowei, et al. "Unified Vision-Language Pre-Training for Image Captioning and VQA." _AAAI_. 2020.
2. Yao, Ting, et al. "Hierarchy parsing for image captioning." _Proceedings of the IEEE International Conference on Computer Vision_. 2019.
3. Wang, Li, et al. "Show, Recall, and Tell: Image Captioning with Recall Mechanism." _AAAI_. 2020.
4. Kim, Dong-Jin, et al. "Image Captioning with Very Scarce Supervised Data: Adversarial Semi-Supervised Learning Approach." _arXiv preprint arXiv:1909.02201_ \(2019\).
5. Zhang, Wei, et al. "Learning Long-and Short-Term User Literal-Preference with Multimodal Hierarchical Transformer Network for Personalized Image Caption." _AAAI_. 2020.

