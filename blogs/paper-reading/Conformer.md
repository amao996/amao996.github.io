## Conformer: Convolution-augmented Transformer for Speech Recognition

INTERSPEECH 2020

### Abstract

Transformer擅长捕捉基于内容的全局交互但不擅长捕捉局部特征，而CNN擅长利用局部特征，然而需要更多更深的层去捕捉全局信息。因此本文提出CNN增强的transformer模型Conformer，将CNN融入到transformer中，得到的很好的实验效果。

### Architecture



<div align=center><img src="https://amao996.github.io/blogs/paper-reading/imgs/Conformer/model.png" width="  "></div><br>

#### Multi-Headed Self-Attention Module

<div align=center><img src="https://amao996.github.io/blogs/paper-reading/imgs/Conformer/fig3.png" width="  "></div><br>

#### Convolution Module

<div align=center><img src="https://amao996.github.io/blogs/paper-reading/imgs/Conformer/fig2.png" width="  "></div><br>

#### Feed Forward Module

<div align=center><img src="https://amao996.github.io/blogs/paper-reading/imgs/Conformer/fig4.png" width="  "></div><br>

#### Conformer Block