# 经典方案概要复现

> 本部分针对图像描述基于深度学习的方法中最为朴素、经典的论文《 Show, attend and tell: Neural image caption generation with visual attention》进行复现。
>
> 该论文是基于论文《show and tell: a neural image caption generator》而增加了注意力机制，大大的提高了相关评测指标，是此领域一个里程碑工作。
>
> **本文件仅仅针对在复现过程中需要用到的重要功能进行了代码展示，目的在于使读者了解代码功能与复现框架，熟悉代码风格，然后尽力搭建自己的复现模型。读者直接根据这些代码整合出可以运行的项目会有较大难度（此处并未给出所有代码），后续在读者自行尝试后会给出完整代码供再次阅读和运行**

## 论文模型

​ 本文最重要的贡献就是应用了两种注意力机制在图像描述上，分别称为软注意力（soft attention）和硬注意力（hard attention）。

​ 软硬注意力的区别在于，对于某特征向量，hard attention要么对他的权重只有0或者1 这两个选项。而soft attention则是从0到1 的变量。本次仅以soft 注意力为代表进行实现，因为这种实现应用更加的广泛。

![&#x56FE;1&#xFF1A; &#x6A21;&#x578B;&#x6982;&#x89C8;&#x56FE;&#xFF08;&#x56FE;&#x6E90;&#xFF1A;&#x89C1;&#x6587;&#x672B;&#x53C2;&#x8003;&#x8D44;&#x6599;1&#xFF09;](http://resource.mahc.host/img/figure2.png)

​ 本文模型主要由三部分组成，第一部分是图像特征的抽取，本部分主要采用VGG来提取。模型第二部分对图像特征进行关注。对抽取出的图像特征的每个权重的计算过程，见图3：

![&#x56FE;2&#xFF1A;&#x6CE8;&#x610F;&#x529B;&#x6A21;&#x5757;&#x6743;&#x91CD;&#x53C2;&#x6570;&#x7684;&#x8BA1;&#x7B97;&#xFF08;&#x56FE;&#x6E90;&#xFF1A;&#x89C1;&#x6587;&#x672B;&#x53C2;&#x8003;&#x8D44;&#x6599;1&#xFF09;](http://resource.mahc.host/img/figure3.png)

在注意力参数计算出来以后，在对图像特征求加权和即可得图像得上下文表示向量，记作z.

​ 模型第三部分将得到得上下文向量z送入LSTM中进行结果预测，最终得到结果。

![&#x56FE;3&#xFF1A;&#x6CE8;&#x610F;&#x529B;&#x6A21;&#x578B;&#x793A;&#x610F;&#x56FE;&#xFF08;&#x56FE;&#x6E90;&#xFF1A;&#x89C1;&#x6587;&#x672B;&#x53C2;&#x8003;&#x8D44;&#x6599;1&#xFF09;](http://resource.mahc.host/img/figure4.png)

## 模型复现

### 实验数据

​ 本文所用的实验数据为MS COCO2014 ，与论文中一致，不同的是采用Karpathy \(Karpathy et al. Deep visual-semantic alignments for generating image descriptions. \)分割方法，即验证集和数据集各采用5000张图像，其余数据全部作为训练集。

### 核心代码

​ 本部分仅展示主要功能代码，详细请申请查看原文件。相关代码大部分来源于开源工作的二次改进。

#### 数据预处理

* 数据集Karpathy分割（常用）

  > 希望读者通过这个小功能文件，学会对数据集进行操作

  ```python
  # # Karpathy Split for MS-COCO Dataset
  # 合并原始train val集以获得更多的训练数据，保留val中的5000个作为新val，保留test中的5000个作为新test
  # 生成三个集合的json文件
  import json
  from random import shuffle, seed

  seed(21)  # Make it reproducible ,not vital to read here

  num_val = 5000
  num_test = 5000

  val = json.load(open('../data/annotations/captions_train2014.json', 'r'))
  train = json.load(open('../data/annotations/captions_val2014.json', 'r'))

  # Merge together
  imgs = val['images'] + train['images']
  annots = val['annotations'] + train['annotations']

  shuffle(imgs)

  # Split into val, test, train
  dataset = {}
  dataset['val'] = imgs[:num_val]  # img[k]包括某图片的全部信息，如图像路径和图片id
  dataset['test'] = imgs[num_val: num_val + num_test]
  dataset['train'] = imgs[num_val + num_test:]

  # Group by image ids
  itoa = {}  # 以image_id查找annots信息的词典，包括全部数据
  for a in annots:
      imgid = a['image_id']
      if not imgid in itoa: itoa[imgid] = []
      itoa[imgid].append(a)

  json_data = {}
  info = train['info']
  licenses = train['licenses']

  split = ['val', 'test', 'train']

  for subset in split:

      # images存储所有图片的img信息，annotations存储annots的信息
      json_data[subset] = {'type': 'caption', 'info': info, 'licenses': licenses,
                           'images': [], 'annotations': []}

      for img in dataset[subset]:
          img_id = img['id']
          anns = itoa[img_id]

          json_data[subset]['images'].append(img)
          json_data[subset]['annotations'].extend(anns)

      # 写为三个json文件
      json.dump(json_data[subset], open('../data/karpathy_split_' + subset + '.json', 'w'))

  print('*** Complete ***')
  ```

* 生成词典

  ```python
  import nltk
  import pickle
  import argparse

  from collections import Counter
  from pycocotools.coco import COCO

  class Vocabulary(object):
      # 单词表
      def __init__(self):
          self.word2idx = {}
          self.idx2word = {}
          self.idx = 0

      def add_word(self, word):
          if not word in self.word2idx:
              self.word2idx[word] = self.idx
              self.idx2word[self.idx] = word
              self.idx += 1
      def __call__(self, word):

          # use <unk> to replace unknown word (用<unk>代替未登录词)
          if not word in self.word2idx:
              return self.word2idx['<unk>']
          return self.word2idx[word]

      def __len__(self):
          return len(self.word2idx)

  def build_vocab(cap_path, threshold):
      # build a new vocabulary
      #新建一个单词表#

      coco = COCO(cap_path)
      counter = Counter()
      ids = coco.anns.keys()

      for id in ids:
          caption = str(coco.anns[id]['caption'])
          tokens = nltk.tokenize.word_tokenize(caption.lower())
          counter.update(tokens)

      # filter out words less than threshold 过滤掉频率小于门限值的单词
      words = [word for word, cnt in counter.items() if cnt >= threshold]

      vocab = Vocabulary()
      vocab.add_word('<pad>')
      vocab.add_word('<start>')
      vocab.add_word('<end>')
      vocab.add_word('<unk>')

      for word in words:
          vocab.add_word(word)
      return vocab

  def main(args):

      vocab = build_vocab(args.cap_path, args.threshold)
      vocab_path = args.vocab_path

      # save the vocabulary to pkl format (把单词表保存成pkl格式)
      with open(vocab_path, 'wb') as f:
          pickle.dump(vocab, f)

      print("*** vocabulary size ：{} ***".format(len(vocab)))

  if __name__ == '__main__':
      parser = argparse.ArgumentParser()

      parser.add_argument('--cap_path', type=str, default='../data/annotations/captions_train2014.json',
                          help='caption json file (caption的json文件路径) ')
      parser.add_argument('--vocab_path', type=str, default='../data/annotations/vocab.pkl',
                          help='path of vocabulary (单词表保存路径)')
      parser.add_argument('--threshold', type=int, default=5,
                          help='threshold of word frequence 单词频率门限值')
      args = parser.parse_args()
      main(args)
  ```

#### 质量评测

* 调用第三方实现评测

  ```python
  from pycocotools.coco import COCO
  from pycocoevalcap.eval import COCOEvalCap

  def coco_eval(ref_json, sen_json, debug=False):
      #compute coco metrics#

      coco = COCO(ref_json)#获取val_jason 文件
      cocoresult = coco.loadRes(sen_json)#输入生成结果的文件，返回一个结果的api
      cocoEval = COCOEvalCap(coco, cocoresult)
      if debug == True:
          cocoEval.params['image_id'] = cocoresult.getImgIds() # using it when you debug
      cocoEval.evaluate()
      return cocoEval.eval.items()
  ```

#### 模型

* VGG编码器

  ```python
  class Encoder(nn.Module):
    # encoder of cnn to represent images#

    def __init__(self, type='vgg19_fn'):
      super().__init__()
        self.cnn, self.avgpool = self.cre_cnn(type)

    def forward(self, image):
        batch_size = image.size(0)
        features = self.cnn(image)  # (b,512,14,14)
        fea_vec = self.avgpool(features).view(batch_size, -1)  # (b,512)
        fea_maps = features.permute(0, 2, 3, 1).view(batch_size, 196, -1)#14*14=196
        return fea_vec, fea_maps
  # 下载模型
    def cre_cnn(self, type):
        #select cnn model#
        if type == 'vgg19_fn':
            # (b, 512, 14, 14)
            print("Downloading pretrained vgg19_fn from torchvsion.models...")
            cnn = models.vgg19_bn(pretrained=True)
            print("Done.")
            modules = list(list(cnn.children())[:-2][0].children())[:-1]
            for p in cnn.parameters():
                p.requires_grad = False
            return nn.Sequential(*modules), nn.AvgPool2d(14)
  ```

* Soft Attention

  ```python
  class Attention(nn.Module):
    #Attention module#
    def __init__(self, conf):
        super().__init__()
        self.fea_att = nn.Linear(conf['model']['embed_dim'], conf['model']['att_dim'])
        self.hid_att = nn.Linear(conf['model']['hid_dim'], conf['model']['att_dim'])
        self.att = nn.Linear(conf['model']['att_dim'], 1)
        print("Init Attention Done.")

    def forward(self, fea_maps, hidden):
        fea_att = self.fea_att(fea_maps)  # (b,196,512) -> (b,196,100)
        hid_att = self.hid_att(hidden)  # (b,512) -> (b,100)
        fusion = fea_att + hid_att.unsqueeze(1)  # (b,49,100)
        att = self.att(torch.relu(fusion)).squeeze(2)  # eq4: eti = fatt(ai,ht-1) (b,196)
        alpha = torch.softmax(att, 1)  # eq5: softmax
        z = (fea_maps * alpha.unsqueeze(2)).sum(1)  # (b,512)
        return z, alpha
  ```

* LSTM解码器（继承Decoder再开发）

  ```python
  class SoftDecoder(Decoder):#继承本地实现的decoder

    def __init__(self, conf):
        print("SoftDecoder initialize...")
        super(SoftDecoder, self). __init__(conf)
        self.attention = Attention(conf)
        # generate a scalar beta to control rate of attention (Don't use it)
        self.fb = nn.Linear(conf['model']['hid_dim'], 1)
        print("Init SoftDecoder done.")

    def forward(self, fea_vec, fea_maps, cap, cap_len):
        batch_size = fea_vec.size(0)
        h, c = self.fea_init_state(fea_vec)
        vocab_size = self.vocab_size
        embeddings = self.embed(cap)#获取caption的编码矩阵
        weights = torch.zeros(batch_size, max(cap_len), vocab_size).to(device)
        alphas = torch.zeros(batch_size, max(cap_len), 196).to(device)
        betas = torch.zeros(batch_size, max(cap_len), 1).to(device)
        for t in range(max(cap_len)):
            z, alpha = self.attention(fea_maps, h) #att (b,512) z加权特征，alpha权重系数
            beta = torch.sigmoid(self.fb(h))
            z = beta * z
            h, c = self.lstmcell(torch.cat([embeddings[:, t, :], z], 1),(h, c))
            weight = self.fc(self.dropout(h))
            weights[:, t, :] = weight
            alphas[:, t, :] = alpha
            betas [:, t ,:] = beta
        return weights, alphas, betas
  ```

* Decoder

  > 此类实现很多基础函数，可被调用，提高代码利用率

  ```python
  import pickle
  import torch
  import torch.nn as nn
  import copy
  import time
  import torchvision.models as models

  from caption_utils.vocab import Vocabulary
  device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')

  class Decoder(nn.Module):
      # based decoder#
      def __init__(self, conf):
          super(Decoder, self).__init__()

          self.conf = conf

          with open(conf['load']['vocab_path'], 'rb') as f:
              self.vocab = pickle.load(f)
          self.vocab_size = len(self.vocab)
          print("Loading vocab...",self.vocab_size)

          # embedding (vocab size -> embedding dimension)
          print("Init embedding matrix for vocab.")
          self.embed = nn.Embedding(self.vocab_size, conf['model']['embed_dim'])

          #这里的2* 是什么意思 =》 图片特征+词语特征
          self.lstmcell = nn.LSTMCell(2*self.conf['model']['embed_dim'], conf['model']['hid_dim'])

          # select vocabulary (embedding dimension -> vocab size)
          self.fc = nn.Linear(conf['model']['hid_dim'], self.vocab_size)
          self.dropout = nn.Dropout(0.5)

          # init hidden and cell of lstm
          self.init_h = nn.Linear(conf['model']['fea_dim'], conf['model']['hid_dim'])
          self.init_c = nn.Linear(conf['model']['fea_dim'], conf['model']['hid_dim'])

          self.init_weight()
          print("Init Decoder done.")

      def init_weight(self):
          #initialize the weights of embedding and FC with a uniform distribution of - 0.1 to 0.1#
          self.embed.weight.data.uniform_(-0.1, 0.1)
          self.fc.bias.data.fill_(0)
          self.fc.weight.data.uniform_(-0.1, 0.1)

      def fea_init_state(self, fea_vec):
          #use image features to initiate state of LSTM#
          h = self.init_h(fea_vec)
          c = self.init_c(fea_vec)
          return h, c

      def beam_search(self, fea_vec, fea_maps, num):
          # a case of beam search#
          h, c = self.fea_init_state(fea_vec)
          sentences = [[[1], 1]]
          h_cells = [0]
          h_cells[0] = (h, c)

          # extra variables
          extra_vars = self.extra_var()
          # generate sentence in maximal length
          for i in range(self.conf['model']['fixed_len'] - 1):
              sen_size = len(sentences)
              # candidate sequence and state
              can_sen = [copy.deepcopy(sen) for sen in sentences * num]
              can_hc = [copy.deepcopy(0) for sen in sentences * num]
              can_vars =[ ]
              for var in extra_vars:
                  can_vars.append([copy.deepcopy(v) for v in var * num])

              for j in range(sen_size):
                  probs, preds_id, step_var, h, c= self.step_generate(fea_maps, sentences, h_cells, num, j)

                  for k in range(num):
                      # save the word id and state (把对应的单词id和状态保存到候选句子和状态中)
                      word_id = int(preds_id[k])
                      can_sen[j + sen_size * k][0].append(word_id)    # word id
                      can_sen[j + sen_size * k][1] *= float(probs[0][word_id])    # probability
                      can_hc[j + sen_size * k] = (h, c)
                      for n, can_var in enumerate(can_vars):
                          can_var[j + sen_size * k].append(step_var[n].detach().cpu().numpy())

              # sort by probability (对概率按照降序进行排序，取索引)
              max_ids = sorted(range(len(can_sen)), key=lambda k: can_sen[k][1], reverse=True)[:num]
              # select the No.k candidate sentences and states of 取前num个概率的候选句子和状态
              sentences = [can_sen[max_id] for max_id in max_ids]
              h_cells = [can_hc[max_id] for max_id in max_ids]
              for n,extra_var in enumerate(extra_vars):
                   extra_vars[n]= [can_vars[n][max_id] for max_id in max_ids]
              # stop beam search if the maximal probability sentence have the end signal

              # todo:  problem: how to handle it if the maximal probability sentence not have the end signal,
              # todo: but another sentence that is not the maximal probability sentence have it.
              # todo: 问题：如果最大概率的句子还没生成完，但是有一个句子已经生成了结束符号，但是排在后面，这种情况应该怎么处理
              if sentences[0][0][-1] == 2:
                  break

          # exchange word id to word]
          sentence = []
          ids = []  # including <start>
          for id in sentences[0][0]:
              word = self.vocab.idx2word[id]
              ids.append(id)
              if word == '<start>':
                  continue
              if word == '<end>':
                  break
              sentence.append(word)
          extra_var = [var for var[0] in extra_vars]
          return sentence, extra_var

      def step_generate(self, fea_maps, sentences, h_cells, num, j):
          #rewrite this method when using different model，used in beam_search#
          inp = torch.Tensor([sentences[j][0][-1]]).long().to(device)
          inp = self.embed(inp)
          h_cell = h_cells[j]

          z, alpha = self.attention(fea_maps, h_cell[0])
          beta = torch.sigmoid(self.fb(h_cell[0]))
          z = beta * z
          h, c = self.lstmcell(torch.cat([inp, z], 1), h_cell)
          # compute probability (计算概率)
          probs = torch.softmax(self.fc(h), 1)
          # select the words of No.k maximal probability 取前num个概率最大的单词
          preds_id = torch.argsort(probs, 1, descending=True)[0][:num]
          step_var = [alpha, beta]

          return probs, preds_id, step_var, h, c
  ```

#### 训练

* 加载训练数据\(训练时被调用\)

  ```python
  import os
  import nltk
  import torch
  import pickle
  import torch.utils.data as data

  from PIL import Image
  from pycocotools.coco import COCO
  from caption_utils.utils import train_transform, val_transform
  from caption_utils.vocab import Vocabulary
  class Trainset(data.Dataset):
      #Built coco dataset in pytorch dataset format#
      def __init__(self, conf):
          ##
          self.img_dir = conf['load']['img_dir']
          self.coco = COCO(conf['load']['train_json'])
          self.ann_ids = list(self.coco.anns.keys())
          vocab_path = conf['load']['vocab_path']
          with open(vocab_path, 'rb') as f:
              self.vocab = pickle.load(f)
          self.conf = conf
          self.transform = train_transform('soft',need_aug=False)#对图像进行维度平滑处理

      def __getitem__(self, index):
          img_dir = self.img_dir
          coco = self.coco
          vocab = self.vocab
          conf = self.conf
          ann_id = self.ann_ids[index]

          cap = coco.anns[ann_id]['caption']
          img_id = coco.anns[ann_id]['image_id']
          path = coco.loadImgs(img_id)[0]['file_name']

          image = Image.open(os.path.join(img_dir, path)).convert('RGB')
          if self.transform is not None:
              image = self.transform(image)

          cap, cap_len = fix_length(cap, vocab, conf['model']['fixed_len'], conf['model']['need_fixed'])

          return image, cap, cap_len
      def __len__(self):
          return len(self.ann_ids)

  def fix_length(cap, vocab, fixed_len, need_fixed='T'):
      #make caption fixed length#

      tokens = nltk.tokenize.word_tokenize(str(cap).lower())
      cap = []
      cap.append(vocab('<start>'))
      cap.extend([vocab(token) for token in tokens])
      cap.append(vocab('<end>'))
      cap_len = len(cap)
      if need_fixed == 'T':
          cap_tensor = torch.zeros(fixed_len).long()
          if cap_len <= fixed_len:
              cap_tensor[:cap_len] = torch.Tensor(cap)
          else:
              cap_tensor[:cap_len] = torch.Tensor(cap[:fixed_len])
              cap_len = fixed_len
      else:
          cap_tensor = torch.zeros(cap_len).long()
          cap_tensor[:] = torch.Tensor(cap)

      return cap_tensor, cap_len

  def train_collate_fn(data):
      #according to length of captions to sort#
      #按照caption的长度排序#

      data.sort(key=lambda x: x[2], reverse=True)
      image, cap, cap_len = zip(*data)
      image = torch.stack(image, 0)
      cap_t = torch.zeros(len(cap), max(cap_len)).long()
      for i, c in enumerate(cap):
          end = cap_len[i]
          cap_t[i, :end] = c[:end]

      return image, cap_t, cap_len

  class Valset(data.Dataset):
      #Load val set of coco caption#

      def __init__(self, conf):
          self.img_dir = conf['load']['img_dir']
          self.coco = COCO(conf['load']['val_json'])
          self.img_ids = list(self.coco.imgs.keys())
          self.transform = val_transform('soft')

      def __getitem__(self, index):
          img_dir = self.img_dir
          coco = self.coco
          img_id = self.img_ids[index]
          path = coco.loadImgs(img_id)[0]['file_name']
          img_path = os.path.join(img_dir, path)
          image = Image.open(img_path).convert('RGB')
          if self.transform is not None:
              image = self.transform(image)

          return image, img_id, img_path

      def __len__(self):
          return len(self.img_ids)

  class Testset(data.Dataset):
      #Load val set of coco caption#

      def __init__(self, conf):
          self.img_dir = conf['load']['img_dir']
          self.coco = COCO(conf['load']['test_json'])
          self.img_ids = list(self.coco.imgs.keys())
          self.transform = val_transform('soft')

      def __getitem__(self, index):
          img_dir = self.img_dir
          coco = self.coco
          img_id = self.img_ids[index]
          path = coco.loadImgs(img_id)[0]['file_name']
          img_path = os.path.join(img_dir, path)
          image = Image.open(img_path).convert('RGB')
          if self.transform is not None:
              image = self.transform(image)
          # print("image shape in test after transform:",image.shape)
          return image, img_id, img_path

      def __len__(self):
          return len(self.img_ids)

  def data_load(conf):

      if conf['model']['model_name'] == 'SCST':
          print("SCST load train set")
          trainset = TrainsetRL(conf)
          train_loader =  data.DataLoader(dataset=trainset,
                                    batch_size=conf['loader']['batch_size'],
                                    shuffle=True,
                                    num_workers=conf['loader']['num_workers'],
                                  #   collate_fn=train_collate_fn,
                                    drop_last=True)
      else:
          trainset = Trainset(conf)
          train_loader =  data.DataLoader(dataset=trainset,
                                    batch_size=conf['loader']['batch_size'],
                                    shuffle=True,
                                    num_workers=conf['loader']['num_workers'],
                                    collate_fn=train_collate_fn,
                                    drop_last=True)
      valset = Valset(conf)
      testset = Testset(conf)
      print("Init train_loader done.")
      val_loader = data.DataLoader(dataset=valset,
                                    batch_size=1,
                                    shuffle=False,
                                    num_workers=conf['loader']['num_workers'],
                                    )

      test_loader =  data.DataLoader(dataset=testset,
                                    batch_size=1,
                                    shuffle=False,
                                    num_workers=conf['loader']['num_workers'],
                                    )

      return train_loader, val_loader, test_loader
  ```

* 保存训练数据

  ```python
  import os
  import json
  import torch
  import pandas as pd

  def save_sen(sentece, epoch, save_dir):
      #save epoch sentences in val set(json format)#
      #save test sentences if epoch is test#
      if not os.path.exists(save_dir):
          os.makedirs(save_dir)

      save_path = os.path.join(save_dir, '{}.json'.format(epoch))
      with open(save_path, 'w') as f:
          json.dump(sentece, f)

      return save_path

  def save_train_inf(results, loss, epoch, save_path):
      #save epoch scores#
      if epoch == 1:
          metrics_list = ['epoch', 'loss','Bleu_1', 'Bleu_2', 'Bleu_3', 'Bleu_4', 'METEOR', 'ROUGE_L', 'CIDEr', 'SPICE']
          metrics_file = pd.DataFrame(columns=metrics_list)
          metrics_file.to_csv(save_path, index=0)

      metrics_file = pd.read_csv(save_path)
      score_dict = {'epoch': epoch}
      #对所有的数据进行精度截取
      for metric, score in results:
          score_dict[metric] = round(score, 3)
      score_dict['loss'] = round(loss, 3)

      metrics_file = metrics_file.append(score_dict, ignore_index=True)
      metrics_file.to_csv(save_path, index=0)

      return score_dict

  def save_best_model(model, optimizer, epoch, best_score_dict, best_epoch, save_path):
      print("Saving best model...")
      # save the best model#
      if not os.path.exists(save_path):
          os.makedirs(save_path)
      save_path = os.path.join(save_path,'best_model.tar')
      torch.save({
          'model': model.state_dict(),
          'optimizer': optimizer.state_dict(),
          'epoch': epoch,
          'best_score_dict': best_score_dict,
          'best_epoch': best_epoch,
      }, save_path)
      print("Saved.")
  ```

* 训练（总管调度）

  > 此 类\(class Train\)实现训练所需相关函数，可被调用开始训练,认真阅读此文件（不要纠结于具体实现，仅通过函数名知道其功能即可）可了解一个模型的训练全过程，对于整体把控很有好处

  ```python
  # 这是一个很重要的训练文件，可以通过继承省去很多不必要的工作
  import os
  import torch
  import pandas as pd
  from tqdm import tqdm
  from torch.nn.utils.rnn import pack_padded_sequence

  from caption_utils.data_load.soft_load import data_load
  from caption_utils.save import save_sen, save_train_inf, save_best_model
  from caption_utils.metrics import coco_eval
  device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')

  from PIL import Image
  from caption_utils.utils import val_transform
  import json

  class Train():

      def __init__(self, conf, model):

          self.conf = conf #配置文件
          self.model = model #模型实例
          self.loss_fn = self.loss_fn()   #cross entropy
          self.optimizer = self.optim(model, conf)    # Adam and lr
          self.train_data, self.val_data, self.test_data = self.load_data()#get all data laoder,not data
          self.predictTransform = val_transform('soft')

      def train(self, debug=False):

          best_score = -0.1#changed by jason from 0 to -0.1
          best_epoch = 0

          for epoch in range(1, 1000):

              # fine tune cnn
              if epoch == self.conf['model']['ft_epoch']:#开始finetune训练CNN
                  print('*** fine tune cnn ***')
                  # load best model within 20 epochs
                  self.model.load_state_dict((torch.load(self.conf['load']['best_model'])['model']))
                  self.optimizer = torch.optim.Adam(
                      [{'params': self.model.decoder.parameters(), 'lr': float(self.conf['model']['de_lr'])},
                       {'params': self.model.encoder.parameters(), 'lr': float(self.conf['model']['en_lr'])}, ],
                      betas=(0.8, 0.999))
                  self.model.encoder.fine_tune()  #把14层(根据不同的模型有不同的参数设置,NIC 7)以后的节点把requires_grad 设置为True

              #TODO 更新学习率
              #scheduler = lr_scheduler.ReduceLROnPlateau(optimizer, 'max', factor=0.8, patience=5, verbose=True)

              print('*** epoch:{} ***'.format(epoch))

              # train
              #每次调用main_loop,就是一个epoch
              loss = self.main_loop(debug)#debug is False by default. 返回每个batch的平均loss
              # evaluate
              #一个epoch过后进行性能测试，返回各项结果
              result = self.eval_val(epoch, debug)

              #保存到csv 文件中，返回经过精度截取的分数
              score_dict = self.save_train_inf(result,  loss, epoch)
              print("Epoch:{} Call Save model...".format(epoch))
              #保存模型，其中还保存了epoch的各项分数，返回经过判断后的最好结果，tag 硬编码为True，jason 修改了存储代码，可以返回false
              best_score, best_epoch, tag = self.save_model(score_dict, best_score, best_epoch, epoch)

              if tag == True:#若连续10轮分数没有上涨就停止
                  print("Training Done.")
                  break

      def load_data(self):

          train_data, val_data, test_data = data_load(self.conf)
          print("Init all data_loader done.")

          return train_data, val_data, test_data

      def main_loop(self, debug):
          #training loop#
          print("Enter main Loop....")
          epoch_loss = 0
          # train loop
          self.model.train()#启用batchnormalize 和 dropout
          print("train data iter...tqdm...")
          for step, (image, cap, cap_len) in tqdm(enumerate(self.train_data)):

              # print(cap[0])    #[batch_size,len],这里的caption就是各个词语的数字表示，没有用pad 进行填充
              # print("step{},image_size:{},cap_size{},cap_len{}",step,image.shape,cap.shape,cap_len.shape)
              image = image.to(device)
              cap = cap.to(device)
              cap_len = [len - 1 for len in cap_len]
              target = pack_padded_sequence(cap[:, 1:], cap_len, batch_first=True)[0]
              print("After Padded,target(caption) size:{}".format(target.size()))

              #把数据已经在模型里过了一遍，得到了loss，算是一个batch的，其中还有一个句子的alpha_loss
              loss = self.model_prop(image, cap, cap_len, target)
              self.back_prop(loss)#反向传播更新，梯度裁剪
              epoch_loss += loss.item()

              if debug == True:

                  if step > 10:
                      print("debug={},step={},break!",debug,step)
                      break

          return epoch_loss / (step+1)

      def loss_fn(self, type='CE'):
          #loss function#

          if type == 'CE':
              return torch.nn.CrossEntropyLoss().to(device)

      def optim(self, model, conf, type='Adam'):
          #select optimizer#

          if type == 'Adam':
              return torch.optim.Adam(model.decoder.parameters(), lr=float(conf['model']['lr']))

      def model_prop(self, image, cap, cap_len, target):
          #method of model propagate#

          batch_size = len(image)
          #送入Encoder，得到两类图像的特征向量，再送到解码器中，返回这三个值，是一句话中各个权重，详见class SoftAtt(nn.Module):的forward函数
          weight, alpha, beta = self.model(image, cap, cap_len)#weight就是预测出来的值
          # print("=======weight ====",type(weight),weight.shape)
          weight = pack_padded_sequence(weight, cap_len, batch_first=True)[0]
          loss = self.loss_fn(weight, target) #计算交叉熵
          alpha_loss = torch.sum(torch.pow((1 - torch.sum(alpha, 1)), 2)) / batch_size#torch.sum(input,1)按行求和，这个loss 不太懂
          loss += alpha_loss
          return loss

      def back_prop(self, loss):
          #update model parameters#

          self.model.zero_grad()
          torch.nn.utils.clip_grad_norm_(self.model.decoder.parameters(),  self.conf['model']['grad_clip'])#梯度裁剪
          loss.backward()
          self.optimizer.step()

      def eval_val(self, epoch, debug):
          #using val dataset to evaluate model performance#
          self.model.eval()#没有batchnormalization和dropout
          sentence_list = []
          for step, (image, img_id, img_path) in tqdm(enumerate(self.val_data)):#val加载的时候batch_size=1
              image = image.to(device)
              img_id = img_id[0]
              # 在各自模型的model 的类中定义 generate
              sentence = self.model.generate(image, beam_num=self.conf['model']['beam_num'])
              sentence = ' '.join(sentence)
              item = {'image_id': int(img_id), 'caption': sentence}
              sentence_list.append(item)
              if debug == True:
                  if step > 10:
                      break

          print('*** compute scores ***')
          #存储一个epoch的所有val图片生成的句子，也包含相关的图像信息，返回的是文件存储地址
          sen_json = save_sen(sentence_list, epoch, self.conf['save']['sen_dir'])
          result = coco_eval(self.conf['load']['val_json'], sen_json, debug)#通过工具，获取各项评测结果,传进去的是路径

          return result

      # 在main 中调用
      def eval_test(self, debug=False):
          print("train done,start to test perfermence...")
          #using val dataset to evaluate model performance
          self.model.load_state_dict(torch.load(self.conf['load']['best_model'])['model'])
          self.model.eval()
          sentence_list = []
          print("generate test set sentences...")
          for step, (image, img_id, img_path) in tqdm(enumerate(self.test_data)):
              # print("eval image shape after transform:",image.shape)
              image = image.to(device)
              img_id = img_id[0]
              sentence = self.model.generate(image, beam_num=self.conf['model']['beam_num'], type='val')
              sentence = ' '.join(sentence)
              item = {'image_id': int(img_id), 'caption': sentence}
              sentence_list.append(item)
              if debug == True:
                  if step > 10:
                      break

          print('*** compute test set scores ***')
          sen_json = save_sen(sentence_list, 'test', self.conf['save']['sen_dir'])
          result = coco_eval(self.conf['load']['test_json'], sen_json, debug)

          metrics_list = ['Bleu_1', 'Bleu_2', 'Bleu_3', 'Bleu_4', 'METEOR', 'ROUGE_L', 'CIDEr',
                          'SPICE']
          metrics_file = pd.DataFrame(columns=metrics_list)

          score_dict = {}
          for metric, score in result:
              score_dict[metric] = round(score, 3)

          metrics_file = metrics_file.append(score_dict, ignore_index=True)
          if not os.path.exists(self.conf['save']['test_dir']):
              os.mkdir(self.conf['save']['test_dir'])
          metrics_file.to_csv(self.conf['save']['test_result'], index=0)

          print('*** complete evaluating test set,everything is Done. ***')

      def save_train_inf(self,result, loss, epoch):
          #save training loss and evaluating result#

          return save_train_inf(result, loss, epoch, self.conf['save']['result'])

      def save_model(self,score_dict, best_score, best_epoch, epoch):
          #save best model on val dataset#
          epoch_score = score_dict['CIDEr']

          print("best_score:",best_score)
          print("epoch_score:",epoch_score)
          # save best model
          #TODO :如果相等，应该选择其他指标尽可能高的，
          if best_score < epoch_score:
              best_score_dict = score_dict
              best_score = epoch_score
              best_epoch = epoch
              save_best_model(self.model, self.optimizer, epoch, best_score_dict, best_epoch, self.conf['save']['checkpoint'])

          if (epoch - best_epoch) > 10:
              print(f'total epoch:{epoch} ')
              print(f'complete training best epoch:{best_epoch}, best CIDEr:{best_score}')

              return best_score, best_epoch, True

          return best_score, best_epoch, False #change True to False by jason
  ```

* main 函数（入口）

  ```python
  class SotfTrain(Train): #继承已实现的train，此类写法可快速切换到新的模型实现

      def __init__(self, conf,model):
          super(SotfTrain, self).__init__(conf, model)

  def main(args):

      #获取学习率等相关参数设定
      with open('./config.yml', 'r') as f:
          conf = yaml.load(f, Loader=yaml.FullLoader)
      # fix random seed，这里设置了4个种子
      set_seed(conf['seed'])

      softAtt = SoftAtt(conf).to(device)#干了不少事，把模型基本上实例化了，还继承了caption_utils 中的模型
      print("Init Encoder,decoder and attention done.")
      softTrain = SotfTrain(conf, softAtt)#虚晃，直接进入caption_utils 中的train，实例化了train
      if args.type == 'train':
          # training
          softTrain.train(args.debug)

      # evaluate test set
      if args.type == 'eval':
          softTrain.eval_test(args.debug)
      print("Everything is Done!!!!!")
  ```

## 复现结果

表1：复现结果对比

| 模型 | Bleu\_1 | Bleu\_2 | Bleu\_3 | Bleu\_4 | METEOR | ROUGE\_L | CIDEr | SPICE |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Source | 70.7 | 49.2 | 34.4 | 24.3 | 23.90 | — | — | — |
| Ours | 71.5 | 54.5 | 41.0 | 31.0 | 25.2 | 53.1 | 95.5 | 18.2 |

​ 表1 是本次复现的结果对比，`Source` 表示论文中的实验结果，`—`表示无法得知，`Ours` 表示复现结果，可以看到指标有很大的提升，原因主要有两个：一是采用现在通用的Karpathy分割方法后，训练数据大大增加了。二是因为时间跨度等因素，很多 trick 更好用了，并且词汇表采用10117大小而非论文中的10000.

### 思考

​ 本次复现的模型是Encoder-Decoder框架下结合注意力机制的经典之作，后来的文章在此基础上不断的改进模型，亦或者是修改编码器，以获得更好抽取图像特征或者caption的编码，指标也越来越高。

​ 然而，指标越来越高并不意味着质量越来越好。例如自从两阶段训练法问世以来，几乎所有的模型都要经过后来的强化训练阶段，指标的增长变得不再明显，有趣的是有的论文每次以上升0.1个指标点而声称达到了SOTA效果，不乏有灌水的嫌疑！

​ 并且，越来越多的人提出指标高了，生成的句子质量却变低了。基于这个现象，我针对指标是否仍有参考价值作了一个小实验进行验证并尝试进行了分析，具体见指标评价思考。

## 参考资料

1. 博客：[图像理解之show attend and tell](https://blog.csdn.net/shenxiaolu1984/article/details/51493673)
2. Vinyals, O., et al. Show and tell: A neural image caption generator. CVPR. 2015.
3. Xu, K., et al. Show, attend and tell: Neural image caption generation with visual attention. in International conference on machine learning. 2015.
4. Karpathy, A. and L. Fei-Fei. Deep visual-semantic alignments for generating image descriptions. CVPR. 2015.

