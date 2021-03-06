# 数据集介绍

> 本节对图像描述中常用的数据集做简单的介绍。
>
> 图像描述中常用的数据集有 MS COCO、Flickr30K、Flickr8K。 其他数据集还有谷歌提出来的CC（Conceptual Captions） 等，读者可自行查找。

## COCO

### 背景

​ MS COCO 是论文《Microsoft COCO: Common Objects in Context》的简写，我们也常常称之为COCO数据集。

​ COCO数据集起源于2014年，主要是由微软主导而标注的，与ImageNet类似，是计算机视觉领域发展的中流砥柱。COCO数据集不仅在图像描述领域有着十分重要的应用，在其他任务例如目标检测、语义分割上也很广泛的应用。

​ 此数据集的目标在于场景理解，图像多为生活场景图，详细介绍见官网首页：[https://cocodataset.org/\#home](https://cocodataset.org/#home)

### 实用介绍

#### 下载

* 下载地址
  * 官网： [https://cocodataset.org/\#download](https://cocodataset.org/#download)
  * 国内：[https://www.functionweb.tk/?/coco2014/](https://www.functionweb.tk/?/coco2014/)
* 下载文件

  > 目前COCO数据集主要分为2014版和2017版本，历史版本之间的区别可在下载页面看到更新日志。
  >
  > 若选择2014版本，则需要下载的文件有五个,分别是：
  >
  > 2014 Train images：训练集的图像压缩包，解压后名称为 train2014.zip
  >
  > 2014 Val images：验证集图像压缩包，解压后名称为 val2014.zip
  >
  > 2014 Test images：测试集图像压缩包，解压后名称为 test2014.zip
  >
  > 2014 Train/Val annotations：训练集、验证集图像的人工标注信息，含图像描述、目标检测等，解压后名称为 annotations\_trainval2014.zip
  >
  > 2014 Tesing Image Info:测试集图像的相关信息，例如图像大小等，无人工标注信息

#### 相关信息

* 哪些人工标注信息？
  * 目标检测、关键点检测、物体分割、多边形分割、图像描述（编者注：文件上分为三类：目标实列、关键节点、图像描述）
* 这些标注的信息形式？
  * 一张图像有5个对其描述的句子（笔者注：5个为做少，也有6个或更多）
  * 图像中每一个物体的检测框和其所属类别
  * 图像中每个物体的mask 信息（语义分割）
  * 图像中每个物体的关键节点信息
* 图像中的物体共有多少类？
  * 包括 人、狗、猫、自行车等91个生活常见类别
* 数据集的大小？
  * 训练集：82783
  * 验证集：40504
  * 测试集：40775

#### 怎么调取数据？

​ COCO官方给了十分方便的操作接口，第三方库为 pycocotools，下面做简单介绍

* 安装：
  * Linux： `pip install cython`   `pip --timeout=200 --no-cache-dir install pycocotools-fix`
  * windows:  参考教程 : [https://www.cnblogs.com/masbay/p/10727280.html](https://www.cnblogs.com/masbay/p/10727280.html)
* 常用接口
  * 官方教程： [https://github.com/dengdan/coco/blob/master/PythonAPI/pycocoDemo.ipynb](https://github.com/dengdan/coco/blob/master/PythonAPI/pycocoDemo.ipynb)
  * | API | 含义 |
    | :--- | :--- |
    | coco = COCO\(json\) | 实例化，必做 |
    | ids = list\(coco.anns.keys\(\)\) | 获得所有caption的ID，么有list也行 |
    | img\_ids = list\(coco.imgs.keys\(\)\) | 获取所有图像的ID |
    | cat\_IDs = coco.getCatIds\(\) | 获取所有的类别ID |
    | cats = coco.loadCats\(cat\_IDS\) | 获取某类别ID的名称，内部元素格式：{'supercategory': 'food', 'id': 58, 'name': 'hot dog'} ，cat\['name'\] =  hot dog |
    | caption =coco.anns\[ann\_id\]\['caption'\] | 获取某captionID内部的caption字段 |
    | img\_id = coco.anns\[ann\_id\]\['image\_id'\] | 获取某captionID的图像ID字段 |
    | path = coco.loadImgs\(img\_id\)\[0\]\['file\_name'\] | 获取某图片的名称 |
    | anns = coco.loadAnns\(annIds\) | 获取标注ID的内容 |
    | caps\_ids  = coco.getAnnIds\(imgIds=image\_id\[i\],  iscrowd=None\) | 获取某图像ID的所有标注ID |
    | cocoresult = coco.loadRes\(sen\_json\) | \#输入生成结果的文件，返回一个结果的api |

## Flickr

### 介绍

​ Flickr 数据集一般有两个，分别是 Flickr8K 和 Flickr30K ，其中 Flicker30K训练集28000，验证测试集分别1K。Flickr8K中有训练集6K，验证集测试分别1K。更详细的信息，请访问官方网址： [http://shannon.cs.illinois.edu/DenotationGraph/](http://shannon.cs.illinois.edu/DenotationGraph/)

#### 实用信息

1. 下载： [http://shannon.cs.illinois.edu/DenotationGraph/data/index.html](http://shannon.cs.illinois.edu/DenotationGraph/data/index.html) （可能需要在官网填写申请）
2. 数据集结构：
   * flickr30k-images.tar ： 存放图像
   * flickr30k.tar.gz ：存放图像的标注信息
3. 每张图片也有5个句子进行描述，数据集中共有158915 句话
4. 使用：

   ```python
   import pandas as pd
   annotations = pd.read_table('results_20130124.token', sep='\t', header=None,
                               names=['image', 'caption'])
   print(annotations['caption'])
   ```

## 参考资料

1. Lin, T.-Y., et al. Microsoft coco: Common objects in context. in European conference on computer vision. 2014. Springer.
2. [https://blog.csdn.net/gaoyueace/article/details/80564642](https://blog.csdn.net/gaoyueace/article/details/80564642)
3. [https://blog.csdn.net/missyougoon/article/details/90256608](https://blog.csdn.net/missyougoon/article/details/90256608)

