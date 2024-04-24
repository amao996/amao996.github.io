## RAFT: Recurrent All-Pairs Field Transforms for Optical Flow

Zachary Teed and Jia Deng
ECCV2020 best paper

### Abstract

提出了Recurrent All-Pairs Field Transforms(RAFT)， 一个光流估计的深度神经网络。RAFT 提取像素级的特征，为所有像素建立多尺度 4D 关联信息，通过查找4D关联信息，循环迭代的更新光流场。
<br><div align=center><img src="https://amao996.github.io/blogs/paper-reading/imgs/RAFT/RAFT-model.png" width="  "></div>

由特征编码器、相关联层、GRU更新层组成

### feature encoder

使用了两个共享权值的CNN，从输入的两张图片中提取像素特征，输出为原输入的1/8倍的特征图。同样的CNN架构也用于上下文网络(Context network)，它只从第一张图像生成特征。在归一化方法上只有一个区别——特征提取器使用实例归一化，而上下文网络使用批处理归一化。
<br><div align=center><img src="https://amao996.github.io/blogs/paper-reading/imgs/RAFT/feature-encoder.png" width="  "></div>

### Computing Visual Similarity

#### Correlation Pyramid

首先基于两个图片的特征图计算correlation volume C

<br><div align=center><img src="https://amao996.github.io/blogs/paper-reading/imgs/RAFT/corr1.png" width="  "></div>

然后通过对C使用不同尺度的池化得到correlation volume，构成correlation pyramid（后两层进行pooling，前两层维度不变）,也就是一个四维张量C，它提供了关于大小像素位移的关键信息。因为C非常大，所以进行降维产生四个不同维度的张量，这样操作可以保证同时捕捉到较大和较小的像素位移。
<br><div align=center><img src="https://amao996.github.io/blogs/paper-reading/imgs/RAFT/pyramid.png" width="  "></div>

#### correlation lookup

定义了一个查询算子，通过从相关金字塔中索引来生成特征图。对于第一个图片中的像素点查找pyramid中对应特征，将frame1图像的点根据确定的光流场映射到frame2，得到对应坐标，邻域表示为下述公式，其中r为搜索半径，把四个层提取到的特征concat到一个特征中
<br><div align=center><img src="https://amao996.github.io/blogs/paper-reading/imgs/RAFT/corr2.png" width="600"></div>

### GRU iterative update

迭代更新基于一个GRU，因为图像特征是2维的，所以分别在水平和竖直两个方向做GRU。
<br><div align=center><img src="https://amao996.github.io/blogs/paper-reading/imgs/RAFT/gru.png" width="600"></div>


输入为迭代预测的光流，相关特征，上下文特征；输出$$\Delta _ { f }$$，则当前迭代下预测的光流就是$$f _ { k + 1 } = f _ { k + 1 } + \Delta _ { f } $$

### Upsample

GRU单元输出的光流分辨率为初始图像的1/8，因此作者提出了两种不同的上采样方法来匹配真值分辨率。第一种是光流的双线性插值。它是一种简单快速的方法，但这种方法的质量不如凸上采样(Convex Upsampling)。

凸上采样(Convex Upsampling)方法表明，全分辨率光流是GRU单元预测的3x3加权网格的凸组合。8 倍图像上采样意味着必须将 1 个像素扩展为 64(8x8) 个像素。凸上采样模块由两个卷积层和末端的softmax激活来预测上采样光流预测中每个新像素的H / 8 × W / 8 × ( 8 × 8 × 9 )掩码。现在，上采样图像上的每个像素都是之前粗糙分辨率像素的凸组合，由预测掩码加权，系数为w1到w9

<br><div align=center><img src="https://amao996.github.io/blogs/paper-reading/imgs/RAFT/upsample.png" width=""></div>

损失函数：L1损失，总损失是真实值与上采样预测之间每个循环块输出的损失之和。加权系数来自于指数计算
<br><div align=center><img src="https://amao996.github.io/blogs/paper-reading/imgs/RAFT/loss.png" width=""></div>

### Innovation

- 引入了motion feature，而motion feature的计算通过金字塔4D关系矩阵均匀采样得来。
- 引入了GRU概念进行迭代优化
- RAFT维护和更新单个固定高分辨率的光流场,而不是coarse-to-fine，首先在低分辨率下估计光流，然后上采样到高分辨率和并细化。