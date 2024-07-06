其他大佬整理的：

https://zhuanlan.zhihu.com/p/421160574



### 先验概率&后验概率

P(A|B)是**已知B发生后A的条件概率**，也由于得自B的取值而被称作**A的后验概率，posterior probability**。posterior的意思是：coming after in time，即在之后发生的。我们的关注目标是A，且A是在B发生之后发生的，所以P(A|B)叫做A的posterior probability。

P(A)是**A的先验概率，prior probability**。prior的意思是：happening or existing before sth else。先发生，所以叫prior probability，即A先发生，它不考虑任何B方面的因素。

贝叶斯法则：
$$
P ( A _ { i } | B ) = \frac { P ( B | A _ { i } ) P ( A _ { i } ) } { \sum _ { j } P ( B | A _ { j } ) P ( A _ { j } ) }
$$
其中$$P ( B | A _ { i } )$$有时被称作**似然度。**贝叶斯法则可以概括为：**后验概率 = (似然度 \* 先验概率)/标准化常量**，也就是说，**后验概率与先验概率和似然度的乘积成正比**。

**贝叶斯分类法的基本思想**的是把计算后验概率的问题转化为计算先验概率的问题