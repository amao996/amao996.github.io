## OVANet: One-vs-All Network for Universal Domain Adaptation

ICCV2021

### Abstract

通用领域自适应UNDA旨在解决两个数据集之间的领域转变和种类转变，主要挑战在于拒绝源数据中不存在但在目标数据中存在的“未知”类。现有方法基于验证或预定义的未知样本比例手动设置阈值来拒绝未知样本，本文提出一种使用源样本来学习阈值并将其适用于目标域的方法。作者的想法是，在源领域中，最小的类间距离应成为在目标中决定“已知”还是“未知”的良好阈值。为了学习类间距离和类内距离，使用带标签的源数据为每个类训练一个一对所有(one-vs-all)的分类器。然后，通过最小化类熵来将开放集分类器适应于目标域。

<div align=center>
<img src="https://amao996.github.io/blogs/paper-reading/imgs/OVANet/model.png" width="  ">
</div><br>