## RAFT-Stereo: Multilevel Recurrent Field Transforms for Stereo Matching

Lahav Lipson 、 ZacharyTeed、Jia Deng

3DV 2021 best paper

### Abstract

提出了一个基于RAFT的新型立体校正深度架构，比较新颖的是提出了多级卷积GRUs(multi-level convolutional GRUs)，效果很好，泛化性和实时推理能力都不错。另外这个模型用于双目图像估计视差图，而不是光流。

<div align=center>
<img src="https://amao996.github.io/blogs/paper-reading/imgs/RAFT-Stereo/model.png" width="  ">
</div><br>

由特征编码器、相关联层、GRU更新层组成

### feature encoder

与RAFT不同的是feature encoder中使用instance normalization，context encoder中使用batch normalization，context feature作为GRU模块的初始化隐藏状态

### Computing Visual Similarity

与RAFT不同的是构建3D correlation volume而不是4D，原因是本文中输入图像像素的位移只在水平方向上发生，而RAFT中像素位移发生在水平和竖直两个方向（rectified stereo）

<div align=center>
<img src="https://amao996.github.io/blogs/paper-reading/imgs/RAFT-Stereo/corr1.png" width="500">
</div><br>

#### correlation pyramid

前两层维度不变，只对最后一个维度进行pooling操作，目的与RAFT相同，保证同时捕捉到较大和较小的像素位移。

#### correlation lookup

使用一个类似于RAFT中定义的查找算子与线性插值来实现检索。给定当前估计的视差, 我们可以在correlation volume中反向寻找对应像素位置处的代价元素, 在每个层级的correlation volume中构建1D的网格来限定查找范围, 然后将不同层级的1D网格元素拼接形成一个单独的feature map.

<div align=center>
<img src="https://amao996.github.io/blogs/paper-reading/imgs/RAFT-Stereo/lookup.png" width="600">
</div><br>

### Multi-Level Update Operator

RAFT的更新模块运行在固定的高分辨率尺度上，会导致卷积核感受野增长不明显，提出multi-resolution update operator（适用于无纹理区域较大，局部信息较少的场景），同步更新分辨率的1/8, 1/16, 1/32的feature map，多个GRU单元之间交叉互联, 交错使用隐藏状态, 但是查找correlation volume和更新最终视差始终在最高分辨率那层的GRU单元进行。

<div align=center>
<img src="https://amao996.github.io/blogs/paper-reading/imgs/RAFT-Stereo/gru.png" width="  600">
</div><br>

dk表示第k次迭代得到视差图d。一次迭代分别进行小中大三个分辨率的更新，隐层也包含三个分辨率，信息在三种分辨率（3个GRU模块）的传递经过上下采样，先从小分辨率开始，得到的结果输入到中分辨率，最后到大分辨率，大分辨率同时输入了查表得到的相关性特征，并在最大分辨率更新视差增量，从而得到新一次迭代的视差图。

### Slow-Fast GRU

小分辨率更新多次再更新一次大分辨率，为了保持精度下尽可能减少高分辨更新次数，可使得运行时间缩短，性能略有下降。

### Loss

与RAFT类似，L1损失，总损失是真实值与上采样预测之间每个循环块输出的损失之和。

<div align=center>
<img src="https://amao996.github.io/blogs/paper-reading/imgs/RAFT-Stereo/loss.png" width="500">
</div><br>

### Innovation

- 将光流预测的模型用于双目视差估计，与传统视差估计不同
- 提出的多级卷积GRUs效果较好

