在带时间序列的任务场景中，输入和输出数据在不同例子中有不同长度，并且标准神经网络并不共享从文本的不同位置学习到的特征、没办法体现出时序上的前因后果。

## RNN

与全连接相比多了一个h0（也就是a0）隐藏状态，整个结构共享权值，只有一个隐状态更新公式。

<div align=center>
<img src="https://amao996.github.io/blogs/ml_and_dl/img/rnn/rnn1.png" width="  ">
</div><br>

优势：输入可以有序，上一个隐藏层的信息可以传递到下一个隐藏层

劣势：因为权重共享，因此梯度在反向传播过程中不断连乘，要么越来越大要么越来越小，所以存在梯度消失和梯度爆炸，难以捕捉长时间的依赖关系。

## LSTM

设计记忆细胞，具备选择性记忆的功能，可以选择记忆重要信息，过滤掉噪声信息，减轻记忆负担。引入输入门，遗忘门，输出门，能够较好解决RNN问题，但是结构复杂。

<div align=center>
<img src="https://amao996.github.io/blogs/ml_and_dl/img/rnn/lstm1.png" width="  ">
</div><br>

输入门：控制输入x和当前计算的状态更新到记忆单元的程度大小

遗忘门：决定从细胞状态中丢弃什么信息，控制输入x和上一层隐藏层输出h被遗忘的程度大小。

内部记忆单元：将 

输出门：控制输入x和当前输出取决于当前记忆单元的程度大小

<div align=center>
<img src="https://amao996.github.io/blogs/ml_and_dl/img/rnn/lstm2.png" width="  ">
</div><br>

## GRU