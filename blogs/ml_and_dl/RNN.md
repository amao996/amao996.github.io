在带时间序列的任务场景中，输入和输出数据在不同例子中有不同长度，并且标准神经网络并不共享从文本的不同位置学习到的特征、没办法体现出时序上的前因后果。

## RNN

与全连接相比多了一个h0（也就是a0）隐藏状态，整个结构共享权值，只有一个隐状态更新公式。每时刻的隐藏状态只依赖于前一时刻的隐藏状态和当前时刻的输入。

<div align=center>
<img src="https://amao996.github.io/blogs/ml_and_dl/img/rnn/rnn1.png" width="  ">
</div><br>

优势：输入可以有序，上一个隐藏层的信息可以传递到下一个隐藏层。

劣势：因为权重共享，因此梯度在反向传播过程中不断连乘，要么越来越大要么越来越小，所以存在梯度消失和梯度爆炸，难以捕捉长时间的依赖关系。

## LSTM

设计记忆细胞，具备选择性记忆的功能，可以选择记忆重要信息，过滤掉噪声信息，减轻记忆负担。引入输入门，遗忘门，输出门，能够较好解决RNN问题，但是结构复杂。

<div align=center>
<img src="https://amao996.github.io/blogs/ml_and_dl/img/rnn/lstm1.png" width="  ">
</div><br>

<div align=center>
<img src="https://amao996.github.io/blogs/ml_and_dl/img/rnn/lstm2.png" width="  ">
</div><br>

记忆细胞：给予了LSTM选择记忆功能，使得其有能力自由选择每个时间步里面记忆的内容。

<div align=center>
<img src="https://amao996.github.io/blogs/ml_and_dl/img/rnn/lstm3.png" width="  ">
</div><br>

输入门：决定保留多少当前时刻生成的新记忆

遗忘门：决定保留多少旧记忆

输出门：控制输入x和当前输出取决于当前记忆单元的程度大小，把保留后的记忆套到新的知识得到结果用于验证

## GRU

将LSTM的输入门和遗忘门合并为更新门，将记忆单元与隐藏层合并成重置门，省去输出门，结构相对简单，参数少计算效率高。

输入为当前时刻的输入和上一时刻的隐状态，更新门和重置门的输出是由sigmoid激活函数的两个全连接层给出，将重置门的输出与上一时刻隐状态进行点积加上当前时刻输入通过tanh得到当前时刻的候选隐状态，通过更新门的输出确定最终隐藏状态。

<div align=center>
<img src="https://amao996.github.io/blogs/ml_and_dl/img/rnn/gru.png" width="  ">
</div><br>

重置门：决定保留多少旧记忆

更新门：决定保留多少当前时刻生成的新记忆