---
layout: post
title: 模型压缩
date: 2019-05-30 15:50:01.000000000 +09:00
mermaid: true
---
## Network Pruning （网络剪枝）

从权重和神经元的角度考虑，网络经常是 over-parameterized 的。所以，我们通过剪枝来缩小网络规模。([Yann Le Cun et al., 1990])

### 流程

```mermaid
graph TD;
A(预训练网络_网络较大) --> B[评估重要性];
B --> C[去除部分_网络较小];
C --> D[微调];
D --> E{符合条件吗};
E -->|是|F[更小的网络];
E -->|否|B
```

* 重要性的评估可以有：
1. 权重：L1、L2 ……
2. 神经元：在给定数据集中非零的次数 ……

* 剪枝后，准确率会下降，通过在 **训练集** 上微调来 recover。但是不要一次剪枝过多，要不然会难以 recover。

### 剪枝的原因

既然我们要剪枝，为什么不一开始训练一个小的网络？对这个问题，有一些不同的观点。

1. 众所周知，更小的网络会更难优化。([李宏毅 Deep Learning Theory 2-4: Geometry of Loss Surfaces (Conjecture)])

2. Lottery Ticket Hypothesis

以往我们剪枝后，就重新随机初始化参数，如下图路线 红 -> 紫 -> 绿，但是效果并不好。但是，如果我们剪枝后将原始随机初始化的参数 copy 过来再训练，发现效果还不错。

![](http://ww4.sinaimg.cn/large/006tNc79ly1g4gwoplwqgj314f0u00wx.jpg)

Lottery Ticket Hypothesis ([Jonathan Frankle et al., 2018]) 认为，若将整个大的网络看做多个小的网络，其中有些小网络可以训练，有的无法训练。于是我们 copy 原始参数会一定程度上拷贝成功的经验。

但是，[Zhuang Liu et al., 2018] 对此提出了不同的观点，其使用真正的随机初始化代替 Lottery Ticket Hypothesis 中提到的方法，发现效果并不差：

![](http://ww3.sinaimg.cn/large/006tNc79gy1g4gwpaa62qj30x4088dh8.jpg)

### Practical Issue

* 权值剪枝（ weight pruning ）

![](http://ww4.sinaimg.cn/large/006tNc79gy1g4gwpil5mtj30ty0cg0tm.jpg)

权值剪枝后会使得网络变得不规则，难以实现，也难以加速（因为 GPU 加速也是进行矩阵运算）。

所以，其实权值剪枝工程上会用置零来代替真正的权值剪枝。但是这种情况实际上并没有减小网络规模。

[Wei Wen  et al., 2016] 对其进行评估，效果显示剪枝 90%+ 的情况下，性能只是下降2%

![](http://ww3.sinaimg.cn/large/006tNc79gy1g4gwpr3qh3j30y00b4mye.jpg)

* 神经元剪枝（ Neuron pruning ）

![](http://ww2.sinaimg.cn/large/006tNc79gy1g4gwpyho2gj30ui0c275b.jpg)

很明显，剪枝后是规则的网络模型，容易实现，也易于加速。

## Knowledge Distillation （知识蒸馏）

### 基本框架

[Geoffrey Hinton et al., 2015] 提出 Knowledge Distillation 的框架进行模型压缩。其主要思想是通过一个 Teacher - Student 的结构，将模型集合中的知识提炼到单个模型中。如下图，这个模型让人惊喜的地方在于，即便 smaller network 里边没有接触到 "7" 这个训练样本，也有一定的机会预测出来，因为 teacher 网络中是一个包含概率的网络，这一点可能会被 student 网络学习到。

![](http://ww1.sinaimg.cn/large/006tNc79gy1g4gwqkihzfj30ft08yjrl.jpg)

在 Kaggle 比赛中，我们经常会集成多个模型来进行预测。但是工业上不可能同时跑这么多个模型，Knowledge Distillation 使得 smaller network 一定程度上实现了集成学习。

![](http://ww2.sinaimg.cn/large/006tNc79gy1g4gwqsmrvwj30g8097aaa.jpg)

### Temperature

论文中提到 一个称之为 Temperature 的概念。在有些训练的最后，我们会使用 Sofamax。为了产生更加 soft 的目标概率分布，用了一个 Temperature 对最后一层的输出进行转换，如下：

$$
y_{i}=\frac{\exp \left(x_{i}\right)}{\sum_{j} \exp \left(x_{j}\right)} \underset{T=100}{\longrightarrow} \quad y_{i}=\frac{\exp \left(x_{i} / T\right)}{\sum_{j} \exp \left(x_{j} / T\right)}
$$

比如：

$$
\begin{array}{lll}{x_{1}=100} & {y_{1}=1} & {x_{1} / T=1} & {y_{1}=0.56} \\ {x_{2}=10} & {y_{2} \approx 0} & {x_{2} / T=0.1} & {y_{2}=0.23} \\ {x_{3}=1} & {y_{3} \approx 0} & {x_{3} / T=0.01} & {y_{3}=0.21}\end{array}
$$

另外，[Lei Jimmy Ba  et al., 2015] 凭经验证明浅层前馈网络可以学习以前通过深网学习的复杂函数，并实现以前只有深模型才能实现的精度。此外，在某些情况下，浅层神经网络可以使用与原始深层模型相同数量的参数来学习这些深层函数。 在TIMIT音素识别和CIFAR-10图像识别任务中，可以训练浅层神经网来执行类似于复杂、更深层次的卷积架构。

## Parameter Quantization （参数量化）

1. 使用更少的位数来表示一个值

2. Weight Clustering

通过 Kmeans 之类的算法，将权值聚合。那么我们将可以使用 Table 中的四个数来表达表中的值，比如说橙色表示的几个数都是 (-3.4 -5 .0) / 2 = - 4.2。

![](http://ww2.sinaimg.cn/large/006tNc79gy1g4gwr087ewj30gg04pq33.jpg)

3. 更进一步的，通过哈夫曼编码之类的算法，可以用较少的比特表示频繁的簇，用更多的比特表示稀有的簇

4. Binary Weights

![](http://ww2.sinaimg.cn/large/006tNc79gy1g4gwr81omzj30g3082q33.jpg)

如上图，根据最近的 “点” （ network with binary weights ） 进行更新直到满意。

下图是 Binary Connect 的效果：

![](http://ww2.sinaimg.cn/large/006tNc79gy1g4gwrfizyqj30fq084my1.jpg)

> 参考 [2] [3] [4]

## Architecture Design （结构设计）

从结构设计上入手来压缩模型，可能是业界比较实用的方法，其基本框架如下：

![](http://ww3.sinaimg.cn/large/006tNc79gy1g4gwruacsbj30g409hdfx.jpg)

在两层之间插入一层来达到压缩模型的效果。以标准 CNN 来举个例子：

![](http://ww1.sinaimg.cn/large/006tNc79gy1g4gws3ayqmj30f40bg3zb.jpg)

如上图，我们需要 72 个参数。改进后：

![](http://ww2.sinaimg.cn/large/006tNc79gy1g4gwsbcfxwj30f20b5dg7.jpg)

![](http://ww3.sinaimg.cn/large/006tNc79gy1g4gwslhl2ij30fg0bp74r.jpg)

![](http://ww3.sinaimg.cn/large/006tNc79gy1g4gwsxvsljj30gb0c4wez.jpg)

相关模型参考 [5] [6] [7] [8]

## Dynamic Computation （动态计算）

动态计算的思想是在资源充足的情况下运行比较复杂的、耗资源的模型。资源不充足的情况下减少运算量，先求出堪用的结果。可能的解决方案有：

1. 训练多个模型：这中方式太耗费存储资源（特别是手机之类的设备）

2. 中间层加入分类器，在资源不足的情况下使用前边的分类器，如下图：

![](http://ww3.sinaimg.cn/large/006tNc79gy1g4gwt7d5l4j307w056mx3.jpg)

但是，一般情况下，网络具备泛化能力和底层抽出的简单特征有关，如果在底层加入分类器，相当于迫使其放弃简单特征，这种方式会造成一定的问题。参考 [Gao Huang et al., 2015]

![](http://ww1.sinaimg.cn/large/006tNc79gy1g4gwtg541lj30lp09iwfo.jpg)

![](http://ww3.sinaimg.cn/large/006tNc79gy1g4gwtnjxs4j30lp09st9s.jpg)


## 参考链接

[1]. [台灣大學 李宏毅 《Network Compression》](http://speech.ee.ntu.edu.tw/~tlkagk/courses/ML_2019/Lecture/Small%20(v6).pdf)

[2]. [Binary Connect](https://arxiv.org/abs/1511.00363)

[3]. [Binary Network](https://arxiv.org/abs/1602.02830)

[4]. [XNOR-net](https://arxiv.org/abs/1603.05279)

[5]. [SqueezeNet](https://arxiv.org/abs/1602.07360)

[6]. [MobileNet](https://arxiv.org/abs/1704.04861)
 
[7]. [ShuffleNet](https://arxiv.org/abs/1707.01083)

[8]. [Xception](https://arxiv.org/abs/1610.02357)

[Yann Le Cun et al., 1990]: http://papers.nips.cc/paper/250-optimal-brain-damage.pdf  'LeCun, Yann, John S. Denker, and Sara A. Solla. "Optimal brain damage." Advances in neural information processing systems. 1990.'

[李宏毅 Deep Learning Theory 2-4: Geometry of Loss Surfaces (Conjecture)]: https://www.youtube.com/watch?v=_VuWvQUMQVk 'Deep Learning Theory 2-4: Geometry of Loss Surfaces (Conjecture)'

[Jonathan Frankle et al., 2018]: https://arxiv.org/abs/1803.03635 'Frankle, Jonathan, and Michael Carbin. "The lottery ticket hypothesis: Finding sparse, trainable neural networks." arXiv preprint arXiv:1803.03635 (2018).'

[Zhuang Liu et al., 2018]: https://arxiv.org/abs/1810.05270 'Liu, Zhuang, et al. "Rethinking the value of network pruning." arXiv preprint arXiv:1810.05270 (2018).'

[Wei Wen  et al., 2016]: https://arxiv.org/pdf/1608.03665.pdf 'Wen, Wei, et al. "Learning structured sparsity in deep neural networks." Advances in neural information processing systems. 2016.'

[Geoffrey Hinton et al., 2015]: https://arxiv.org/pdf/1503.02531.pdf 'Hinton, Geoffrey, Oriol Vinyals, and Jeff Dean. "Distilling the knowledge in a neural network." arXiv preprint arXiv:1503.02531 (2015).'

[Lei Jimmy Ba  et al., 2015]: https://arxiv.org/pdf/1312.6184.pdf 'Ba, Jimmy, and Rich Caruana. "Do deep nets really need to be deep?." Advances in neural information processing systems. 2014.'

[Gao Huang et al., 2015]: https://arxiv.org/pdf/1703.09844.pdf 'Huang, Gao, et al. "Multi-scale dense networks for resource efficient image classification." arXiv preprint arXiv:1703.09844 (2017).'
