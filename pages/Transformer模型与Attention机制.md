### 介绍
- Attention Is All You Need是一篇Google提出的将Attention思想发挥到极致的论文。这篇论文中提出一个全新的模型，叫 Transformer，抛弃了以往深度学习任务里面使用到的 CNN 和 RNN ，目前大热的Bert就是基于Transformer构建的，这个模型广泛应用于NLP领域，例如机器翻译，问答系统，文本摘要和语音识别等等方向。
- ### Transformer总体结构
- 和Attention模型一样，Transformer模型中也采用了 encoer-decoder 架构。但其结构相比于Attention更加复杂，论文中encoder层由6个encoder堆叠在一起，decoder层也一样。
  
  ![](https://cdn.nlark.com/yuque/0/2021/png/2713067/1611453077527-a35ca107-0b72-49c0-bfd4-c15b2679134a.png)
- 每一个encoder和decoder的内部简洁版结构：
  
  ![](https://cdn.nlark.com/yuque/0/2021/png/2713067/1611453110306-9dcc2aaa-a07f-46c9-a670-c44dabf56820.png)
- 对于encoder，包含两层，一个self-attention层和一个前馈神经网络，self-attention能帮助当前节点不仅仅只关注当前的词，从而能获取到上下文的语义。
- decoder也包含encoder提到的两层网络，但是在这两层中间还有一层attention层，帮助当前节点获取到当前需要关注的重点内容。
- 现在我们知道了模型的主要组件，接下来我们看下模型的内部细节。首先，模型需要对输入的数据进行一个embedding操作，也可以理解为类似w2c的操作，enmbedding结束之后，输入到encoder层，self-attention处理完数据后把数据送给前馈神经网络，前馈神经网络的计算可以并行，得到的输出会输入到下一个encoder。
  
  ![](https://cdn.nlark.com/yuque/0/2021/png/2713067/1611453486396-d57cf086-33a2-4f06-8d81-29e88b4bb07c.png)
- ### Self-Attention
- 接下来我们详细看一下self-attention，其思想和attention类似，但是self-attention是Transformer用来将其他相关单词的“理解”转换成我们正在处理的单词的一种思路，我们看个例子： The animal didn't cross the street because it was too tired 这里的it到底代表的是animal还是street呢，对于我们来说能很简单的判断出来，但是对于机器来说，是很难判断的，self-attention就能够让机器把it和animal联系起来，接下来我们看下详细的处理过程。
	- 首先，self-attention会计算出三个新的向量，在论文中，向量的维度是512维，我们把这三个向量分别称为Query、Key、Value，这三个向量是用embedding向量与一个矩阵相乘得到的结果，这个矩阵是随机初始化的，维度为（64，512）注意第二个维度需要和embedding的维度一样，其值在BP的过程中会一直进行更新，得到的这三个向量的维度是64低于embedding维度的。
	- ![](https://cdn.nlark.com/yuque/0/2021/png/2713067/1611453614061-5e38414d-d7f8-4b7c-a0b3-dbada2791216.png)
	- 计算self-attention的分数值，该分数值决定了当我们在某个位置encode一个词时，对输入句子的其他部分的关注程度。这个分数值的计算方法是Query与Key做点成，以下图为例，首先我们需要针对Thinking这个词，计算出其他词对于该词的一个分数值，首先是针对于自己本身即q1·k1，然后是针对于第二个词即q1·k2
	- ![](https://cdn.nlark.com/yuque/0/2021/png/2713067/1611453692368-d491aab0-1941-44c7-a9ec-97ca9ac1baa4.png)
	- 接下来，把点成的结果除以一个常数，这里我们除以8，这个值一般是采用上文提到的矩阵的第一个维度的开方即64的开方8，当然也可以选择其他的值，然后把得到的结果做一个softmax的计算。得到的结果即是每个词对于当前位置的词的相关性大小，当然，当前位置的词相关性肯定会会很大
	- ![](https://cdn.nlark.com/yuque/0/2021/png/2713067/1611453720402-69cee2ac-51d7-459d-af77-6120e2d3585f.png)
	- 下一步就是把Value和softmax得到的值进行相乘，并相加，得到的结果即是self-attetion在当前节点的值。
	- ![](https://cdn.nlark.com/yuque/0/2021/png/2713067/1611454077360-5a68967f-77ed-420e-81b4-1de6013f8aaa.png)
	- 在实际的应用场景，为了提高计算速度，我们采用的是矩阵的方式，直接计算出Query, Key, Value的矩阵，然后把embedding的值与三个矩阵直接相乘，把得到的新矩阵Q与K相乘，乘以一个常数，做softmax操作，最后乘上V矩阵
	- 这种通过 query 和 key 的相似性程度来确定 value 的权重分布的方法被称为scaled dot-product attention。
	- ![](https://cdn.nlark.com/yuque/0/2021/png/2713067/1611454155916-81cfd140-56cf-469a-92f2-21999828642b.png)
	- ![](https://cdn.nlark.com/yuque/0/2021/png/2713067/1611454159903-15757957-9e93-491b-bfaf-ce4bc3273276.png)
- ### Multi-Headed Attention
- 这篇论文更牛逼的地方是给self-attention加入了另外一个机制，被称为“multi-headed” attention，该机制理解起来很简单，就是说不仅仅只初始化一组Q、K、V的矩阵，而是初始化多组，tranformer是使用了8组，所以最后得到的结果是8个矩阵。
- ![](https://cdn.nlark.com/yuque/0/2021/png/2713067/1611454371379-54ed8568-86aa-45f3-8ca5-783dfed7e461.png)
- ![](https://cdn.nlark.com/yuque/0/2021/png/2713067/1611454376120-4fbdbf8b-0897-42af-96c2-583e83d9ce0a.png)
- 这给我们留下了一个小的挑战，前馈神经网络没法输入8个矩阵呀，这该怎么办呢？所以我们需要一种方式，把8个矩阵降为1个，首先，我们把8个矩阵连在一起，这样会得到一个大的矩阵，再随机初始化一个矩阵和这个组合好的矩阵相乘，最后得到一个最终的矩阵。
  
  ![](https://cdn.nlark.com/yuque/0/2021/png/2713067/1611454407562-acc22c20-0ede-4d5d-bd73-42402f34ee93.png)
- 这就是multi-headed attention的全部流程了，这里其实已经有很多矩阵了，我们把所有的矩阵放到一张图内看一下总体的流程。
  
  ![](https://cdn.nlark.com/yuque/0/2021/png/2713067/1611454424536-ac1f67b4-156b-44c7-87c6-2eb3f7522acb.png)