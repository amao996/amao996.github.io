## Attention Is All You Need

NIPS 2017

### Abstract

旧问题新技术，提出一种完全基于注意力机制的transformer架构，不使用CNN和RNN，并行度很高。 

### 知识概述

- encoder：将$x = (x1, x2, ... , xn)$（原始输入）映射成$z = (z1, z2, ..., zn)$（机器学习可以理解的向量）

- decoder：给定z，生成一个输出序列$(y1,y2,...ym)$（n和m可以一样长也可以不一样长），只能一个一个生成。

- auto-regressive:过去时刻的输出又作为当前时刻的输入

- attention机制：q、k、v分别表示query、key、value，输出为一个value的加权和，权重等价于query和对应key的相似度

- Scaled Dot-Product Attention：对query和所有的key进行点积得到值，再对点积结果除以$\sqrt{d _ {k}}$，完成scale；在scale之后进行mask，目的是当t时刻的query只能看到k1到kt-1；再通过softmax获得每个value对应的权重，最后和value加权求和得到输出向量。

  <div align=center><img src="https://amao996.github.io/blogs/paper-reading/imgs/Transformer/attention1.png" width="  "></div><br>

  <div align=center><img src="https://amao996.github.io/blogs/paper-reading/imgs/Transformer/math1.png" width="  "></div><br>

  当$d _ {k}$较大时，向量之间的点积结果非常大，会造成softmax函数陷入到梯度很小的区域，不利于反向传播，因此设置缩放因子$\sqrt{d _ {k}}$对点积结果进行尺度化，将其缩小到梯度敏感的区域内。当dk较大时使用Additive attention比较好，Additive attention可以处理q和k不等长的情况。

- Multi-Head Attention：h次机会学习不同的投影方法，分别做点积再拼接到一起做一次投影。

<div align=center><img src="https://amao996.github.io/blogs/paper-reading/imgs/Transformer/attention2.png" width="  "></div><br>

- Positional Encoding：在输入中加入时序信息

## Architecture

<div align=center><img src="https://amao996.github.io/blogs/paper-reading/imgs/Transformer/model.png" width="  "></div><br>

### Encoder

由六个相同的层堆叠而成，每层由一个多头注意力块和一个位置全连接前馈网络组成，在子层之间采用残差连接和正则化。

1. 多头注意力块
2. 位置全连接前馈网络：线性层+ReLU+线性层
3. 残差连接：每个子层的输出为该子层的输入和输出之和，要求每个子层的输入和输出维度相同。（训练的时候可以使梯度直接走捷径反传到最初始层）
4. 正则化(Layer Normalization)：在把数据送入激活函数之前进行，将数据归一化为均值位0，方差为1的数据（加快训练速度，加速收敛的作用，同时也可以避免梯度消失和梯度爆炸的问题）。Batch Normalization更适用于CNN处理图像，Layer Normalization更适用于序列化模型。



### Decoder

由六个相同的层堆叠而成，与encoder相比多一个掩码多头注意力机制，目的是让t时刻不会看到t时刻之后的输入而只能看到之前的输入（也就是过滤掉t时刻之后的数据）。key和value来自于encoder的输出，query来自decoder下一个attention的输入，也就是掩码多头注意力块的输出。