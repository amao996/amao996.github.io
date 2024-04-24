## RAFT-Stereo: Multilevel Recurrent Field Transforms for Stereo Matching

Lahav Lipson 、 ZacharyTeed、Jia Deng

3DV best paper

### Abstract

提出了Recurrent All-Pairs Field Transforms(RAFT)， 一个光流估计的深度神经网络。RAFT 提取像素级的特征，为所有像素建立多尺度 4D 关联信息，通过查找4D关联信息，循环迭代的更新光流场。

<div align=center>
<img src="https://amao996.github.io/blogs/paper-reading/imgs/RAFT-Stereo/model.png" width="  ">
</div><br>

由特征编码器、相关联层、GRU更新层组成

### feature encoder

与RAFT不同的是feature encoder中使用instance normalization，context encoder中使用batch normalization，context feature作为GRU模块的初始化隐藏状态

### Correlation Pyramid

与RAFT不同的是构建3D correlation volume而不是4D，原因是本文中输入图像像素的位移只在水平方向上发生（rectified stereo）

<div align=center>
<img src="https://amao996.github.io/blogs/paper-reading/imgs/RAFT-Stereo/corr1.png" width="  ">
</div><br>

correlation pyramid：只对最后一个维度进行pooling操作

**correlation lookup**：使用一个类似于RAFT中定义的查找算子与线性插值来实现检索。给定当前估计的视差, 我们可以在correlation volume中反向寻找对应像素位置处的代价元素, 在每个层级的correlation volume中构建1D的网格来限定查找范围, 然后将不同层级的1D网格元素拼接形成一个单独的feature map.

<div align=center>
<img src="https://amao996.github.io/blogs/paper-reading/imgs/RAFT-Stereo/lookup.png" width="  ">
</div><br>

### Multi-Level Update Operator

输入：当前估计的视差，相关特征，上下文特征；更新隐藏状态

RAFT的更新模块运行在固定的高分辨率尺度上，会导致卷积核感受野增长不明显，提出multi-resolution update operator（适用于无纹理区域较大，局部信息较少的场景），同步更新分辨率的1/8, 1/16, 1/32的feature map，多个GRU单元之间交叉互联, 交错使用隐藏状态, 但是查找correlation volume和更新最终视差始终在最高分辨率那层的GRU单元进行。

<div align=center>
<img src="https://amao996.github.io/blogs/paper-reading/imgs/RAFT-Stereo/gru.png" width="  ">
</div><br>

Slow-Fast GRU模块（可用可不用）：可使得运行时间缩短，性能略有下降。

损失函数：与RAFT类似

<div align=center>
<img src="https://amao996.github.io/blogs/paper-reading/imgs/RAFT-Stereo/loss.png" width="  ">
</div><br>