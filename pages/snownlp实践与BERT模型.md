##### 前言
- 今天主要就是学习BERT模型以及snownlp关于不同模型的运用。
- ##### snownlp实践
- 这一次主要就是使用snownlp进行对应的改编，以达到需要的效果。
- 这次的训练数据集来自于网上的10000条外卖的数据集。因为是直接从网上下载下来的，因此还没有按照readme的要求。因此首先需要的就是对一个统一的csv文件进行标签分类的操作。
  
  ```
  # encoding:utf-8
  import csv
  
  # 任务：
  # 1. 读取原来test.csv文件中的每一行数据
  # 2. 将读取到的数据进行分类，分为积极评论和消极评论
  
  # 文件基本信息
  base_folder = '../csv_learn/'
  base_filepath = '../test.csv'
  pos_filepath = 'pos.txt'
  neg_filepath = 'neg.txt'
  
  base_file = open(base_filepath, 'r+', encoding='utf-8')
  pos_file = open(base_folder+pos_filepath, 'w', encoding='utf-8')
  neg_file = open(base_folder+neg_filepath, 'w', encoding='utf-8')
  
  csv_file = csv.DictReader(base_file)
  print("csv文件读写工具加载完成。")
  
  # label是表示的这个评论的好坏，0表示的就是消极评论，1表示的是积极评论
  for item in csv_file:
    if item["label"] == '0':
        neg_file.write(item["review"]+'\n')
    elif item["label"] == '1':
        pos_file.write(item["review"]+'\n')
  
  # 文件流的关闭
  base_file.close()
  pos_file.close()
  neg_file.close()
  ```
- 使用分类完成之后的消极和积极评论txt，进行新的训练。
  
  ```
  from snownlp import SnowNLP
  from snownlp import sentiment
  
  base_folder = '../csv_learn/'
  save_marshal_filepath = 'sentiment.marshal'
  pos_filepath = 'pos.txt'
  neg_filepath = 'neg.txt'
  
  sentiment.train(base_folder+neg_filepath,base_folder+pos_filepath)
  sentiment.save(base_folder+save_marshal_filepath)
  ```
- 训练之后，将sentiment中的训练模型替换掉就可以了。替换之后，产生的效果确实比之前的要好不少，但是也仅仅仅限于这一领域内而已。
- ##### snownlp情感分析再理解
- 现假设： 所有的句子可以分类两类：积极态度(c1)、消极态度(c2). 一个句子有特征w1,w2,...,wn，这n个特征相互独立. 对于情感分类任务中特征w1,w2,...,wn分别代表该句子中对应的单词
	- 预测的过程使用到了上述的公式，即
		- ![](https://cdn.nlark.com/yuque/0/2021/png/2713067/1611301906974-9897a745-20c9-4925-b3f7-5dea86884362.png)
	- 对上述的公式简化：
		- ![](https://cdn.nlark.com/yuque/0/2021/png/2713067/1611301923163-529add02-bf15-4064-8b99-2896a003855e.png)
	- 其中，分母中的1可以改写为：
		- ![](https://cdn.nlark.com/yuque/0/2021/png/2713067/1611301945754-e877aef6-f39f-4d90-a82e-2dda9d32d0ae.png)
- #### BERT模型
- ##### 预训练模型
- 假设已有A训练集，先用A对网络进行预训练，在A任务上学会网络参数，然后保存以备后用，当来一个新的任务B，采取相同的网络结构，网络参数初始化的时候可以加载A学习好的参数，其他的高层参数随机初始化，之后用B任务的训练数据来训练网络，当加载的参数保持不变时，称为"frozen"，当加载的参数随着B任务的训练进行不断的改变，称为“fine-tuning”，即更好地把参数进行调整使得更适合当前的B任务
- 优点：当任务B的训练数据较少时，很难很好的训练网络，但是获得了A训练的参数，会比仅仅使用B训练的参数更优
- ##### 预备知识——Seq2Seq
- 在开始讲解Attention之前，我们先简单回顾一下Seq2Seq模型，传统的机器翻译基本都是基于Seq2Seq模型来做的，该模型分为encoder层与decoder层，并均为RNN或RNN的变体构成，如下图所示：
  
  ![](https://cdn.nlark.com/yuque/0/2021/gif/2713067/1611379601129-b9ef9b05-57b6-46a1-9ef1-60a4dc4b488e.gif)
- 在encode阶段，第一个节点输入一个词，之后的节点输入的是下一个词与前一个节点的hidden state，最终encoder会输出一个context，这个context又作为decoder的输入，每经过一个decoder的节点就输出一个翻译后的词，并把decoder的hidden state作为下一层的输入。该模型对于短文本的翻译来说效果很好，但是其也存在一定的缺点，如果文本稍长一些，就很容易丢失文本的一些信息，为了解决这个问题，Attention应运而生。
- ##### Attention机制（注意力机制）
- Attention，正如其名，注意力，该模型在decode阶段，会选择最适合当前节点的context作为输入。Attention与传统的Seq2Seq模型主要有以下两点不同。
	- 1）encoder提供了更多的数据给到decoder，encoder会把所有的节点的hidden state提供给decoder，而不仅仅只是encoder最后一个节点的hidden state。
	- ![](https://cdn.nlark.com/yuque/0/2021/gif/2713067/1611379713323-6b78fe04-5c83-4a07-9736-9dcfdb116f5c.gif)
	- 2）decoder并不是直接把所有encoder提供的hidden state作为输入，而是采取一种选择机制，把最符合当前位置的hidden state选出来，具体的步骤如下
		- 确定哪一个hidden state与当前节点关系最为密切
		- 计算每一个hidden state的分数值（具体怎么计算我们下文讲解）
		- 对每个分数值做一个softmax的计算，这能让相关性高的hidden state的分数值更大，相关性低的hidden state的分数值更低。
- 把每一个encoder节点的hidden states的值与decoder当前节点的上一个节点的hidden state相乘，如上图，h1、h2、h3分别与当前节点的上一节点的hidden state进行相乘(如果是第一个decoder节点，需要随机初始化一个hidden state)，最后会获得三个值，这三个值就是上文提到的hidden state的分数，注意，这个数值对于每一个encoder的节点来说是不一样的，把该分数值进行softmax计算，计算之后的值就是每一个encoder节点的hidden states对于当前节点的权重，把权重与原hidden states相乘并相加，得到的结果即是当前节点的hidden state。可以发现，其实Atttention的关键就是计算这个分值。
  
  ![](https://cdn.nlark.com/yuque/0/2021/gif/2713067/1611379785577-9f202af6-efec-4782-a5fd-4afd6a1fea39.gif)
- 接下来就是decoder层的工作原理了，其具体过程如下：
	- 第一个decoder的节点初始化一个向量，并计算当前节点的hidden state，把该hidden state作为第一个节点的输入，经过RNN节点后得到一个新的hidden state与输出值。注意，这里和Seq2Seq有一个很大的区别，Seq2Seq是直接把输出值作为当前节点的输出，但是Attention会把该值与hidden state做一个连接，并把连接好的值作为context，并送入一个前馈神经网络，最终当前节点的输出内容由该网络决定，重复以上步骤，直到所有decoder的节点都输出相应内容。
	- Attention模型并不只是盲目地将输出的第一个单词与输入的第一个词对齐。实际上，它在训练阶段学习了如何在该语言对中对齐单词(示例中是法语和英语)。Attention函数的本质可以被描述为一个查询（query）到一系列（键key-值value）对的映射。
	- 在计算attention时主要分为三步，第一步是将query和每个key进行相似度计算得到权重，常用的相似度函数有点积，拼接，感知机等；然后第二步一般是使用一个softmax函数对这些权重进行归一化；最后将权重和相应的键值value进行加权求和得到最后的attention。目前在NLP研究中，key和value常常都是同一个，即key=value。
		- ![](https://cdn.nlark.com/yuque/0/2021/png/2713067/1611379888923-6cd83aa1-d9ab-4ae2-857e-d38180522028.png)
-