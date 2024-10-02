## Position Encoding

- 绝对位置编码（absolute PE）：将位置信息加入到输入序列中，相当于引入索引的嵌入。
- 相对位置编码（relative PE）：通过微调自注意力运算过程使其能分辨不同token之间的相对位置。

### APE

在输入序列经过词嵌入后的第$$k$$个token向量中加入位置向量$$p_ {k}$$，其过程等价于首先向输入引入位置索引$$k$$的one hot向量$$p_{k} = x _ {k} + p _ {k}$$，再进行词嵌入，绝对位置编码APE也被称为位置嵌入(Position Embedding)。

从绝对位置编码出发，其形式相当于在输入中加入绝对位置的表示，对应完整自注意力机制运算如下：
$$
q _ { i } = ( x _ { i } + p _ { i } ) W ^ { Q } , k _ { j } = ( x _ { j } + p _ { j } ) W ^ { K } , v _ { j } = ( x _ { j } + p _ { j } ) W ^ { V }
$$

$$
\alpha _ { i j } = s o f t \max \{ ( x _ { i } + p _ { i } ) W ^ { Q } ( ( x _ { j } + p _ { j } ) W ^ { K } ) \} \\ = s o f t \max \{ x _ { i } W ^ { Q } ( W ^ { K } ) ^ { T } x _ { j } ^ { T } + x _ { i } W ^ { Q } ( W ^ { K } ) ^ { T } p _ { j } ^ { T } + p _ { i } W ^ { Q } ( W ^ { K } ) ^ { T } p _ { j } ^ { T } \}
$$

$$
z _ { i } = \sum _ { j = 1 } ^ { n } \alpha _ { i j } ( x _ { j } W ^ { V } + p _ { j } W ^ { V } )
$$

绝对位置编码相当于在自注意力运算中引入了一系列的$$ p _ { i } W ^ { Q } , ( p _ { j } W ^ { K } ) ^ { T } , p _ { j } W ^ { V }$$项，而相对位置编码便是将这些项调整为与相对位置$$(i, j)$$有关的向量。

#### 三角函数位置编码

这种编码是在原transformer模型中使用的，以一维三角函数编码为例：
$$
P E _ { ( p o s , 2 i ) } = \sin ( p o s / 1 0 0 0 0 ^ { 2 i / d } )
$$

$$
P E _ { ( p o s , 2 i + 1) } = \cos ( p o s / 1 0 0 0 0 ^ { 2 i / d } )
$$

其中$$P E _ { ( p o s , 2 i ) }$$和$$P E _ { ( p o s , 2 i+1 ) }$$分别是位置索引$$k$$处的编码向量的第$$2i$$，$$2i+1$$个分量。

### RPE

相对位置编码并不直接建模每个输入token的位置信息，而是在计算注意力矩阵时考虑当前向量与待交互向量的位置的相对距离。

#### 经典相对位置编码

提出自Self-Attention with Relative Position Representations

本方法移除了与$$x_{i}$$的位置编码项$$p _ { i } W ^ { Q } $$相关的项，并将$$x _ {j}$$的位置编码项$$p _ { j } W ^ { V } $$和$$p _ { j } W ^ { K } $$替换为相对位置向量$$ R _ { i , j } ^ { V } , R _ { i , j } ^ { K }$$。

原始自注意力过程：
$$
e _ { i , j } = \frac { ( x _ { i } W ^ { Q } ) ( x _ { j } W ^ { K } ) ^ { T } } { \sqrt { d _ { z } } }
$$

$$
\alpha _ { i , j } = \frac { e x p ( e _ { i , j } ) } { \sum _ { k = 1 } ^ { n } e x p ( e _ { i , k } ) }
$$

$$
z _ { i } = \sum _ { j = 1 } ^ { n } \alpha _ { i , j } ( x _ { j } W ^ { V } )
$$

加入RPE变换的部分：
$$
e _ { i , j } = \frac { ( x _ { i } W ^ { Q } ) ( x _ { j } W ^ { K } + a _ { i , j } ^ { K } ) ^ { T } } { \sqrt { d _ { z } } }
$$

$$
z _ { i } = \sum _ { j = 1 } ^ { n } \alpha _ { i , j } ( x _ { j } W ^ { V } + a _ { i , j } ^ { V } )
$$

也就是一方面在计算attention权重的时候，根据i和j的相对位置关系给定一个RPE；另一方面得到权重计算context vector时候再给一个RPE。

对于$$a _ { i , j },a _ { i , j } ^ { K },a _ { i , j } ^ { V }$$的计算：$$ a _ { i , j } = w _ { c l i p ( j - i , k ) } ^ { K } \in R ^ { d _ { z } }$$，其中$$ c l i p ( j - i , k ) = m a x ( - k , m i n ( k , x ) ) \in [ - k , k ]$$。$$a _ { i , j }$$与i和j的差值有关，同时差值有上限，相离过远时，统一把距离记作k。