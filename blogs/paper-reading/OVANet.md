## OVANet: One-vs-All Network for Universal Domain Adaptation

ICCV2021

### Abstract

通用领域自适应UNDA旨在解决两个数据集之间的领域转变和种类转变，主要挑战在于拒绝源数据中不存在但在目标数据中存在的“未知”类。现有方法基于验证或预定义的未知样本比例手动设置阈值来拒绝未知样本，本文提出一种使用源样本来学习阈值并将其适用于目标域的方法。作者的想法是，在源领域中，最小的类间距离应成为在目标中决定“已知”还是“未知”的良好阈值。为了学习类间距离和类内距离，使用带标签的源数据为每个类训练一个一对所有(one-vs-all)的分类器。然后，通过最小化类熵来将开放集分类器适应于目标域。

<div align=center>
<img src="https://amao996.github.io/blogs/paper-reading/imgs/OVANet/model.png" width="  ">
</div><br>
### Structure

OVANet主要分为两部分，第一部分是使用学习到的源类别之间的距离来识别未知样本；第二部分是通过最小化开放集熵适应目标域。

两个分类器：

- 分类器$$O$$：open-set classifier，检测未知样本，通过标准分类损失来训练以对源样本进行分类，在测试阶段用于识别最近的已知类

- 分类器$$C$$：closed-set classifier，将样本分类到源标签$$L _ {s}$$中，通过HNCS训练，在测试阶段用于确定样本是否属于未知类

| 分类器 | open-set classifier | closed-set classifier  |
| ------ | ------------------- | ---------------------- |
| usage  | 检测未知样本        | 将样本分类到源标签中   |
| train  | 标准分类损失        | HNCS                   |
| test   | 识别最近的已知类    | 确定样本是否属于未知类 |

#### Hard Negative Classifier Sampling (HNCS)



#### Open-set Entropy Minimization (OEM)



### Experiment

#### Evaluation Metric

H分数：是共同类$$acc _ {c}$$和未知类$$acc _ {t} $$的准确性调和平均值
$$
H _ { s c o r e } = \frac { 2 a c c _ { c } \cdot a c c _ { t } } { a c c _ { c } + a c c _ { t } . }
$$
缺点在于已知类和未知类的权重永远相同，当未知实例数量较小时，该度量会过度强调未知类别。

#### Implementation

backbone：ResNet50

Evaluation：VGGNet

