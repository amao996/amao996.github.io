## "GrabCut" ——Interactive Foreground Extraction using Iterated Graph Cuts

### Abstract

从三方面扩展graph-cut方法。1.开发一个优化版本；2.迭代算法被用于简化为给定结果质量所需的用户交互；3.开发了"border matting"算法，可以同时估计物体边界周围的alpha-matte和前景像素的颜色。

### Graph Cut

#### Image Segmentation

图像为$$Z$$，图像分割表示为每个像素处的不透明度$$\alpha$$，对于hard segmentation，$$\alpha$$取值为0或1（0表示background，1表示foreground），参数$$\theta$$描述图像的foreground和background的灰度级分布，由灰度值直方图组成（直接从相应的三元组区域$$T _ {B}, T _ {F}$$收集像素）。
$$
\theta = \{ h ( z ; \alpha ) , \alpha = 0 , 1 \}
$$
分割任务是给定$$Z$$和$$\theta$$推断不透明度$$\alpha$$。

#### Segmentation by energy minimisation

定义能量函数$$E$$，函数的最小值对应于一个好的分割。即，分割的合理程度 = 分割结果的内部合理程度（从灰度分布来看，每个像素的值是否合理） + 分割边缘的合理程度（从边界相邻像素的角度来看，当前划分是否合理）
$$
E ( \alpha , \theta , z ) = U ( \alpha , \theta , z ) + V ( \alpha , z )
$$
其中数据项$$U$$在给定直方图模型$$\theta$$下，估计不透明度$$\alpha$$与数据$$Z$$之间的拟合。
$$
U ( \alpha , \theta , z ) = \sum _ { n } - \log h ( z _ { n } ; \alpha _ { n } )
$$
$$V$$为平滑项，
$$
V ( \alpha , z ) = \gamma \sum _ { ( m , n ) \in C } d i s ( m , n ) ^ { - 1 } \left[ \alpha _ { n } \neq \alpha _ { m } \right] e x p - \beta ( z _ { m } - z _ { n } ) ^ { 2 }
$$
其中$$C$$为相邻像素对的集合，$$dis(m,n)$$是相邻像素对之间的欧几里得距离，$$ \beta = ( 2 < ( z _ { m } - z _ { n } ) ^ { 2 } > ) ^ { - 1 }$$，$$<>$$表示图像样本的期望，$$ \beta$$为，$$\gamma = 50$$。

在能量函数$$E$$定义后，分割可以估计为全局最小值。
$$
\widehat { \alpha } = a r g \min _ { \alpha } E ( \alpha , \theta )
$$
如果仅考虑第一项（数据项），整个分割就变成了基于阈值的图像分割，会跟大津法等的结果类似。第二项（平滑项）的限制，会使分割倾向于使用尽可能少的分割，并尽量在高对比处分割。

整个流程的限制是：1. 算法基于灰度图；2. 需要人工标注至少一个前景点和一个背景点；3. 结果为硬分割结果，未考虑边缘介于0~1之间的透明度。

### GrabCut

三个进展：用高斯混合模型（GMM）代替直方图来代替单色图像模型，以支持彩色图片；替换单词最小cut估计算法；允许不完整标记来减轻用户交互需求。

先获得一个hard segmentation，然后进行border matting。

#### Colour data modelling（构建图）

$$
E ( \alpha , k , \theta , z ) = U ( \alpha , k , \theta, z ) + V ( \alpha , z )
$$

数据项$$U$$考虑到颜色GMM模型，被定义为下式，也就是某个像素属于目标或背景的概率负对数。
$$
U ( \alpha , k , \theta , z ) = \sum _ { n } D ( \alpha _ { n } , k _ { n } , \theta , z _ { n } )
$$
其中$$ D ( \alpha _ { n } , k _ { n } , \theta , z _ { n } ) = - \log p ( z _ { n } | \alpha _ { n } , k _ { n } , \theta ) - \log \pi ( \alpha _ { n } , k _ { n } )$$，$$ p ( · )$$是高斯概率分布，$$ \pi ( · )$$是混合权重系数。因此
$$
D ( a _ { n } , k _ { n } , \theta , z _ { n } ) = - \log \pi ( a _ { n } , k _ { n } ) + \frac { 1 } { 2 } \log d e t \sum ( a _ { n } , k _ { n } ) \\ + \frac { 1 } { 2 } [ z _ { n } - \mu ( a _ { n } , k _ { n } ) ] ^ { T } \sum ( a _ { n } , k _ { n } ) ^ { - 1 } [ z _ { n } - \mu ( a _ { n } , k _ { n } ) ]
$$
模型现在的参数是$$ \theta = \left\{ \pi ( \alpha , k ) , \mu ( \alpha , k ) , \sum ( \alpha , k ) , \alpha = 0 , 1 , k = 1 \ldots K \right\}$$，也就是background和foreground共2K个高斯分量的权重、均值和协方差。

平滑项$$V$$使用欧几里得距离在彩色空间中计算对比度项
$$
V ( \alpha , z ) = \gamma \sum _ { ( m , n ) \in C } \left[ \alpha _ { n } \neq \alpha _ { m } \right] e x p - \beta | | z _ { m } - z _ { n } | | ^ { 2 }
$$

#### 初始化

1. 用户通过直接框选目标来得到一个初始的trimap T，即方框外的像素全部作为背景像素TB，而方框内TU的像素全部作为“可能是目标”的像素。
2. 对TB内的每一像素n，初始化像素n的标签αn=0，即为背景像素；而对TU内的每个像素n，初始化像素n的标签αn=1，即作为“可能是目标”的像素。
3. 经过上面两个步骤，我们就可以分别得到属于目标（αn=1）的一些像素，剩下的为属于背景（αn=0）的像素，这时候，我们就可以通过这个像素来估计目标和背景的GMM了。我们可以通过k-mean算法分别把属于目标和背景的像素聚类为K类，即GMM中的K个高斯模型，这时候GMM中每个高斯模型就具有了一些像素样本集，这时候它的参数均值和协方差就可以通过他们的RGB值估计得到，而该高斯分量的权值可以通过属于该高斯分量的像素个数与总的像素个数的比值来确定。


#### 迭代最小化

第一步：对每个像素分配GMM中的高斯分量（例如像素n是目标像素，那么把像素n的RGB值代入目标GMM中的每一个高斯分量中，概率最大的那个就是最有可能生成n的，也即像素n的第kn个高斯分量）

第二步：学习优化GMM的参数，对于foreground model中的给定GMM组件k，像素子集被定义为$$ F ( k ) = \left\{ z _ { n } : k _ { n } = k  \ and  \ \alpha _ { n } = 1 \right\}$$，使用$$F ( k )$$中像素值的样本均值和协方差来估计$$\mu ( \alpha , k ) , \sum ( \alpha , k )$$，权重为$$ \pi ( \alpha , k ) = | F ( k ) | / \sum _ { k } | F ( k ) |$$，其中|S|表示S集合的大小。

第三步：建立一个图，并求出权值t-link和n-link，然后使用minimum cut进行分割

迭代最小化步骤1到3中的每一 步都可以证明是对总能量E相对于三个变量集k、θ和α的最小化。因此，E单调递减。当E停止显著下降时，自动停止迭代。

#### User Interaction andincomplete trimaps

##### Incomplete trimaps

用户只需要指定background区域$$T _ {B}$$而不是整个trimap，迭代最小化通过允许foreground区域的一些像素具有临时标签（随后可以被撤销），只有background标签$$T _ {B}$$是牢固的。（反过来也可以设置background的临时标签，foreground是牢固的）。初始$$T _ {B}$$标签由用户定义为矩形外部的一圈像素。

##### further user editing

采用刷像素的形式，将指定像素限制为牢固的foreground或background，然后应用minimum cut进行全局优化。

### Transparency

#### Border Matting

#### Foreground estimation