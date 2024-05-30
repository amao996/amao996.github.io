## Learning Optical Flow from Continuous Spike Streams

NeurIPS 2022

### Abstract

提出用于光流估计的Spike2Flow网络，该网络基于differential of spike firing time(DSFT)和空间信息聚合(SIA)的二值脉冲中提取信息，网络架构在RAFT的基础上丰富了一些组件。并构造了一个真实场景的数据集RSSF。

<div align=center>
<img src="https://amao996.github.io/blogs/paper-reading/imgs/spike2flow/model.png" width="  ">
</div><br>
主要分为对脉冲进行时空表征encoder，相关性联合解码decoder两部分。

### Dataset

以Slow Flow数据集为基础生成flow的ground truth，31个train场景，10个test场景。通过RGB照片生成flow作为模型的ground truth。生成方法是融入注意力机制的RAFT模型。

```tex
images -> flow1;flow2;flow3(采用不同的间隔生成)
spike -> dsft
```

### Temporal-Spatial Representation for Spikes

<div align=center>
<img src="https://amao996.github.io/blogs/paper-reading/imgs/spike2flow/encoder.png" width=" ">
</div><br>
#### 融入时间信息(DSFT)

脉冲流中的每个脉冲表示的是积分过程而不是当前时刻的状态，脉冲流中的“1”表示的是累积光子的数量，而不是光子的到达率，使用从二进制脉冲中提取的特征进行匹配可能无法准确反映场景的结构，因此通过发射时间（firing time）来表示脉冲中包含的信息。脉冲中的每个像素被表示为the difference in firing time（当前时刻前后脉冲达到阈值的最小时段）。

<div align=center>
<img src="https://amao996.github.io/blogs/paper-reading/imgs/spike2flow/dsft.png" width=" ">
</div><br>
计算公式如下

<div align=center>
<img src="https://amao996.github.io/blogs/paper-reading/imgs/spike2flow/dsft2.png" width=" ">
</div><br>

其中$$ T _ { n e x t } ( x , t )$$和$$ T _ { pre } ( x , t )$$表示当前时刻后一个和前一个脉冲达到阈值的时刻。

#### 特征提取

特征提取网络与RAFT的相同，对输入进行下采样，提取1/8分辨率的特征图，同样的CNN架构也用于上下文网络(Context network)，它只从第一个输入中生成特征。

<div align=center><img src="https://amao996.github.io/blogs/paper-reading/imgs/RAFT/feature-encoder.png" width="500"></div><br>

#### 融入空间信息(SIA)

通过自注意力机制融入上下文信息得到最终的feature。
$$
F = F _ { p } + s o f t \max ( \theta ( F _ { p } ) \phi ( F _ { p } ) ^ { T } ) \cdot g ( F _ { p } )
$$
其中$$F _ { p }$$表示未经过SIA的特征，$$\theta,\phi,g$$分别对应注意力机制中的query，key，value。

### Joint Decoding of Correlation

<div align=center>
<img src="https://amao996.github.io/blogs/paper-reading/imgs/spike2flow/decoder.png" width=" ">
</div><br>
从相同起始时刻联合解码一系列运动，对于每个解码过程，与RAFT相同，具体如下。

#### Correlation Pyramid

首先基于两个特征图计算correlation volume C

<div align=center><img src="https://amao996.github.io/blogs/paper-reading/imgs/RAFT/corr1.png" width="  "></div><br>

然后通过对C使用不同尺度的池化得到correlation volume，构成correlation pyramid（后两层进行pooling，前两层维度不变）,也就是一个四维张量C，它提供了关于大小像素位移的关键信息。因为C非常大，所以进行降维产生四个不同维度的张量，这样操作可以保证同时捕捉到较大和较小的像素位移。

<div align=center><img src="https://amao996.github.io/blogs/paper-reading/imgs/RAFT/pyramid.png" width="  "></div><br>

#### correlation lookup

对于每个像素而言只需要查找出与他相关的correlation region即可，定义了一个查询算子，通过从相关金字塔中索引来生成特征图。对于第一个图片中的像素点$$ x = ( u , v )$$查找pyramid中对应特征，将frame1图像的点根据当前的光流场估计$$ ( f ^ { 1 } , f ^ { 2 } )$$映射到frame2上的$$ x ^ { \prime } = ( u + f ^ { 1 } ( u ) , v + f ^ { 2 } ( v ) )$$，定义的搜索域表示为下述公式，其中r为搜索半径，把四个层提取到的特征concat到一个特征中

<div align=center><img src="https://amao996.github.io/blogs/paper-reading/imgs/RAFT/corr2.png" width="600"></div><br>

#### GRU iterative update

迭代更新基于一个GRU，因为特征是2维的，所以分别在水平和竖直两个方向做GRU。

<div align=center><img src="https://amao996.github.io/blogs/paper-reading/imgs/RAFT/gru.png" width="600"></div><br>


输入为上一次估计的光流，循环卷积的隐藏状态和相关信息张量；输出 $\Delta _ { f }$，则当前迭代下预测的光流就是 $f _ { k + 1 } = f _ { k + 1 } + \Delta _ { f } $

### Loss

使用average end-point error (AEPE)和percentage of outliers (PO%)作为指标，其中AEPE是预测光流与真实光流的欧氏距离空间平均值；PO%是端点误差同时大于0.5和5 %真实值的像素所占百分比。

### Ablation Study

<div align=center>
<img src="https://amao996.github.io/blogs/paper-reading/imgs/spike2flow/ab.png" width=" ">
</div><br>

消融实验可知，其中DSFT模块是最不可或缺的，SIA和JCD对模型均有适当的提升。

### Conclusion

- 我认为本文主要的创新在于将光流预测网络用于脉冲输入上
- 另外提出了比如SIA，JCD等对模型有适当提升的模块