# Deep Residual Learning for Image Recognition

概述：引入深度残差学习框架，通过**跨层连接（short connections）**，不直接拟合底层映射而是拟合**残差映射**

![image-20240117103206132](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20240117103206132.png)

## Architecture

### Residual Learning

由于当模型层数越多但是训练误差和测试误差都增大的现象（不是由过拟合产生），因此不期望堆叠层近似$H(x)$，而是显式地让这些层近似一个残差函数$F (x):=H (x)-x$。

### short connections

不引入新的参数也不增加模型复杂度。

输入和输出的维度必须相同，如果不同的话可以通过投影来匹配维度。下面的第二个式子是投影的方法。

![image-20240117110522311](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20240117110522311.png)

![image-20240117110537388](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20240117110537388.png)

identity mapping：输入是x，输出也是x，输入和输出一一对应。$f(a)=a$

1*1卷积层：实现特征通道的升降维