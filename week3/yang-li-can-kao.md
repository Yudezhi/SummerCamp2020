# 样例参考

## ~~标注原则~~

### 本文档已过时，请参考 评分细则！！

> $$\color{red}{本文档本周内会根据实践实时更新，请及时查阅；也愿各位读者在标注过程中遇到问题及时反馈，我们及时调整策略}$$

1. 标注需要考虑的因素：

   一个好的标注肯定会考虑很多因素，并且这些因素之间“鱼和熊掌不可兼得”，因此在评价时就要有所取舍。本次我们经过讨论决定，主要从以下三个方面来进行人工评价：

   * 对象个数与完整性 ：是否反映了图像中的主体目标
   * 正确关系个数 ：在给出的句子中，对象之间的关系是否正确
   * 正确属性个数：给出的对象的相关属性（数量、颜色等）是否正确

2. 其他说明：
   * 正确描述一种对象、关系、属性（`或` 关系），得1分，句子整体可以表达图像内容对象+1分
   * 数量词 `a` 由于在数据集中过于普遍，不加分;其他数量词可得分
   * 由于系统初期设计原因，以0.5分代表0分，但1.5分非1分
   * 女人：  对象为人，属性为 女
   * 或对或错：遇到某描述不确定是否正确时标注人根据整体印象+0.5分或1分
   * 不正确：出现不正确的描述时，不加分，不扣分

## 标注示例

**23873**

![image\_id:23873](http://resource.mahc.host/img/image-20200720074241819.png)

人工标注：

![](http://resource.mahc.host/img/image-20200720074407545.png)

⼈为识别图⽚中的内容 **⼀些物体：** tower、clock（严格要求的话，应该是clock tower\)、hotel、other buildings **⼀些场景：** street、city

* 模型一： a large clock tower in the middle of a city.

  > **对象**：clock 、tower、city ，对象分3分。
  >
  > **关系**：in the middle of a city是⽆法从图中看出来的，也不是⼀个合理的推理，所以关系分0.5分
  >
  > **属性**：属性词是large，⽐较合适，属性分1分

* 模型二：a clock on the side of a building on a city street.

  > **对象**：clock，city street，对象分给2分
  >
  > **关系**：on a city street 是个⽐较合理的推理，但是on the side of a building是把tower识别成了building， 这个关系描述的有些冗余，可以给0.5分 所以关系分给1.5分
  >
  > **属性**：city在这⾥可以算是⼀个属性词，1分

* 模型三： a clock tower in the middle of a city.

  > 只是⽐模型⼀少了 large，分析同模型⼀

* 模型四： a clock on the side of a building.

  > ⽐模型⼆少了on a city street，分析同模型⼆

* 模型五： a view of a city with a clock tower in the background.

  > **对象**：city, clock、tower， 对象分3分
  >
  > **关系**：两个关系 with 和 in, 都⽐较合适 关系分2分
  >
  > **属性**：a view of ...，属性分给1分

**18473**

![image-20200720075820867](http://resource.mahc.host/img/image-20200720075820867.png)

人工标注：

![image-20200720075851925](http://resource.mahc.host/img/image-20200720075851925.png)

这个场景的物体⽐较复杂 物体： dog、book、desk、couch、fireplace 等等 场景： room

* 模型一： A living room with a couch and a fireplace .

  > **对象**：living room 、couch 、 fireplace 对象分 3分
  >
  > **关系**：with ，关系分1分
  >
  > **属性**：living ，属性分1分

* 模型二： A living room with a couch and a tv

  > **对象**：living room 、couch 对象分2分
  >
  > **关系**：with，但是tv是错的，关系分给1分（若tv识别成火炉，则此处潜在表示两个关系，可给两分）
  >
  > **属性**：living ，属性分1分

* 模型三：a dog laying on a bed in a room.

  > **对象**：dog、bed、room， 对象分3分
  >
  > **关系**：laying on\(动词关系，给1.5分）、 in 关系分 共计 2.5分
  >
  > **属性**：0.5分

* 模型四：A living room with a couch and a fireplace

  > 同模型一

* 模型五：A living room with a couch and a table .

  > 同模型一

**10858**

![image-20200720081400797](http://resource.mahc.host/img/image-20200720081400797.png)

人工标注：

![image-20200720081436893](http://resource.mahc.host/img/image-20200720081436893.png)

* 模型一： a hot dog covered in foil and mustard .

  > **对象**：hot dog、foil、mustard\(这个不好确定是不是芥末酱\)，2.5或3分
  >
  > **关系**：cover in ，1.5分
  >
  > **属性**： 0.5分

* 模型二：a hot dog with lots of toppings on a foil wrapper .

  > **对象**: hot dog、toppings、foil wrapper 3分
  >
  > **关系**: with 、on ，2分
  >
  > **属性**：lots of ,foil 2分

* 模型三： a close up of a hot dog on a bun

  > **对象**：hot dog 、bun\(和hot dog有冲突成分\) 1.5分
  >
  > **关系**: on ，关系不合适 0.5分
  >
  > **属性**: a close up 1分

* 模型四：a hot dog covered in toppings sitting on aluminum foil.

  > **对象**: hot dog、 toppings、 aluminum foil 3分
  >
  > **关系**：cover in、sitting on 3分
  >
  > **属性**： aluminum, 1分

* 模型五： a hot dog on a bun in a wrapper .

  > **对象**:：hot dog、 bun、 wrapper 3分
  >
  > **关系**： on\(关系不完全正确\)、in 1.5分
  >
  > **属性**：0.5分

**116048**

![image-20200720110023561](http://resource.mahc.host/img/image-20200720110023561.png)

> 理由：模型一、三、四对象正确出现四个，表达正确完整加一分，为5分。属性中预测到是女性为1分。模型五出现三个对象，但表达意思错误，仅得三分。模型二关系down the street 虽然正确，但标注人员认为关系错有违常识，只给0.5分。

