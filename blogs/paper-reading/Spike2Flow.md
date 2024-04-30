## Learning Optical Flow from Continuous Spike Streams

NeurIPS 2022

### Abstract

提出用于光流估计的Spike2Flow网络，该网络基于differential of spike firing time(DSFT)和空间信息聚合的二值脉冲中提取信息，网络架构在RAFT的基础上丰富了一些组件。并构造了一个真实场景的数据集RSSF。

<div align=center>
<img src="https://amao996.github.io/blogs/paper-reading/imgs/spike2flow/model.png" width="  ">
</div><br>

主要分为对脉冲进行时空表征，相关性联合解码两部分

### Temporal-Spatial Representation for Spikes

<div align=center>
<img src="https://amao996.github.io/blogs/paper-reading/imgs/spike2flow/encoder.png" width=" ">
</div><br>

首先对脉冲流进行DSFT表征，用来反映脉冲的发放率，脉冲中的每个像素被表示为the difference in firing time

<div align=center>
<img src="https://amao996.github.io/blogs/paper-reading/imgs/spike2flow/dsft.png" width=" ">
</div><br>

后面的特征提取网络与RAFT的类似；最后进行空间信息的整合(SIA)，通过自注意力机制得到最终的feature

### Joint Decoding of Correlation

<div align=center>
<img src="https://amao996.github.io/blogs/paper-reading/imgs/spike2flow/decoder.png" width=" ">
</div><br>

与RAFT不同的是提出了三个Flow Head以及三个delta_flow，每次迭代对他们都进行更新。

### Loss

使用average end-point error (AEPE)和percentage of outliers (PO%)作为指标，其中AEPE是预测光流与真实光流的欧氏距离空间平均值；PO%是端点误差同时大于0.5和5 %真实值的像素所占百分比。

### Ablation Study

<div align=center>
<img src="https://amao996.github.io/blogs/paper-reading/imgs/spike2flow/ab.png" width=" ">
</div><br>

消融实验可知，其中DSFT模块是最不可或缺的，SIA和JCD对模型均有适当的提升。