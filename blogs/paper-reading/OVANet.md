## OVANet: One-vs-All Network for Universal Domain Adaptation

ICCV2021

### Abstract

通用领域自适应UNDA旨在解决两个数据集之间的领域转变和种类转变，主要挑战在于拒绝源数据中不存在但在目标数据中存在的“未知”类。现有方法基于验证或预定义的未知样本比例手动设置阈值来拒绝未知样本，本文提出一种使用源样本来学习阈值并将其适用于目标域的方法。作者的想法是，在源域中，最小的类间距离应成为在目标域中决定“已知”还是“未知”的良好阈值。为了学习类间距离和类内距离，使用带标签的源数据为每个类训练一个一对所有(one-vs-all)的分类器。然后，通过最小化类熵来将开放集分类器适应于目标域。

<div align=center>
<img src="https://amao996.github.io/blogs/paper-reading/imgs/OVANet/model.png" width="  ">
</div><br>
### Background

#### DA

- closed set DA：源标签集和目标标签集相同
- partial DA：目标标签集是源标签集的子集
- open set DA：源标签集是目标标签集的子集
- open-partial DA：源和目标标签集中存在各自私有的标签集和共同的标签集
- universal DA：源标签域已知，对于任何目标域，如果属于源标签集中的任何类别，则对其进行正确分类，否则标记为unknown类

<div align=center>
<img src="https://amao996.github.io/blogs/paper-reading/imgs/OVANet/DA.png" width="  ">
</div><br>

#### Current methods

目前的方法人工设定了一个阈值来拒绝unknown实例或通过先验知识进行验证。

<div align=center>
<img src="https://amao996.github.io/blogs/paper-reading/imgs/OVANet/intro1.png" width="  ">
</div><br>

- 拒绝一定比例的目标样本，此方法在拒绝比率准确的时候效果较好，但是在没有标记的目标样本情况下估计拒绝比率很难。
- 使用标记的目标样本验证阈值决定unknown：破坏了”没有标记的目标样本“的前提。
- 合成unknown实例：通过可学习模型定义unknown的概念，但是调整生成过程需要标记样本进行验证，否则生成的数据不一定与真实unknown数据相似。

需要实现一个方法既不需要unknown样本的拒绝阈值，也不需要对阈值进行验证。

作者提出能否利用源类之间的类间距离来学习阈值，假设最小的类间距离是一个很好的阈值，用于确定样本是否属于该类别。即如果一个样本与一个类的距离小于边距，则属于该类；反之如果一个样本不属于任何类，则标记为unknown。因此作者提出为每个类训练one-vs-all的分类器，该分类器通过类间距离学习边界。如果所有类别的分类器认为某样本不属于他们，则该样本标记为unknown。

<div align=center>
<img src="https://amao996.github.io/blogs/paper-reading/imgs/OVANet/intro2.png" width="  ">
</div><br>

### 一些符号标记

- $$ D _ { s } = \left\{ ( x _ { i } ^ { s } , y _ { i } ^ { s } ) \right\}$$：源域，包含$$N _ {s}$$个标记的样本
- $$ D _ { t } = \left\{ ( x _ { i } ^ { t } ) \right\}$$：目标域，包含$$N _ {t}$$个未标记的样本
- $$L _ {s}$$：源域的标签集
- $$L _ {t}$$：目标域的标签集
- $$O$$：open-set classifier，使用学习到的类间距离识别未知样本，通过标准分类损失来训练以对源样本进行分类，在测试阶段用于确定样本是否属于未知类
- $$C$$：closed-set classifier，将样本分类到源标签$$L _ {s}$$中，通过HNCS训练，在测试阶段用于识别最近的已知类

### Structure

OVANet主要分为两部分，第一部分是使用学习到的源类别之间的距离来识别未知样本；第二部分是通过最小化开放集熵适应目标域。

train在源域和目标域的并集上，test在目标域上

两个分类器：

| 分类器 | open-set classifier         | closed-set classifier |
| ------ | --------------------------- | --------------------- |
| usage  | 检测未知样本，标记为unknown | 将样本分类到源标签中  |
| train  | HNCS                        | 标准分类损失          |
| test   | 确定样本是否属于未知类      | 识别最近的已知类      |

#### Hard Negative Classifier Sampling (HNCS)

Hard Negative Class：different from the class but similar to it(nearest negative class)

对于每个源样本，更新一个positive和一个hard negative的open-set分类器，以高效学习每个类的类间距离最小值也就是边界。

<div align=center>
<img src="https://amao996.github.io/blogs/paper-reading/imgs/OVANet/HNCS.png" width="  ">
</div><br>

open-set classifier由$$L _ {s}$$个子分类器组成，每个子分类器用于区分样本是否属于该类别并输出一个二维向量，表示该样本属于该类的概率和不属于该类的概率。

前人使用的方法是对于每个子分类器，训练其与$$L _ {s} - 1$$个类别的边界，但是当$$L _ {s}$$很大的时候这种方法并不有效，原因在于open-set classifier可以很容易地对许多negative类进行分类。（比如猫狗龟，要为猫类学习边界应该更关注于与其相似的狗而不是乌龟，上述方法学习到的边界可能在猫和乌龟中间）因此作者提出hard negative class的概念，并为每个样本训练两个分类器，一个是positive类，一个是hard negative类。

对于每个子分类器$$k$$，首先提取特征$$ z ^ { k } = w ^ { k } G _ { \theta } ( x ) \in R ^ { 2 }$$，其中$$G _ { \theta }$$表示特征提取器，$$w ^ { k }$$表示分类器对于类别$$k$$的权重，$$ z ^ { k }$$的两个维度分别表示known和unknown的得分。样本$$x$$属于类别$$k$$的概率表示为$$ p ( \widehat { y } ^ { k } | x ) = \sigma ( z ^ { k } ) _ { 0 }$$，其中$$\sigma$$为激活函数。

<div align=center>
<img src="https://amao996.github.io/blogs/paper-reading/imgs/OVANet/openset_training.png" width="  ">
</div><br>

##### open-set分类的损失函数

$$
\complement _ { o v a } ( x ^ { s } , y ^ { s } ) = - \log ( p ( \widehat { y } ^ { y ^ { s } } | x ^ { s } ) ) - min_ { j \neq y ^ { s } } \log ( 1 - p ( \widehat { y } ^ { j } | x ^ { s } ) )
$$



#### Open-set Entropy Minimization (OEM)

one-vs-all分类器的熵被最小化，允许模型将没标签的目标样本分类到已知类或标记为unknown。



### Experiment

#### Evaluation Metric

H分数：是共同类$$acc _ {c}$$和未知类$$acc _ {t} $$的准确性调和平均值
$$
H _ { s c o r e } = \frac { 2 a c c _ { c } \cdot a c c _ { t } } { a c c _ { c } + a c c _ { t } . }
$$
缺点在于已知类和未知类的权重永远相同，当未知实例数量较小时，该度量会过度强调未知类别。

#### Implementation

backbone:ResNet50

Evaluation:VGGNet

