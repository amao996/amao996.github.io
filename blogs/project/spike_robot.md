## 基于脉冲视觉的移动物体精准抓取研究

### 摘要

​	针对机器人抓取问题，现有研究受限于传统视频帧率低，无法记录光的高速变换过程。调研得知脉冲相机可以捕捉到光的高速变化过程，缺乏基于双目脉冲输入信息提取特征进行抓取姿态预测的模型。因此我们构建高帧率双目脉冲数据集，并提出一个端到端的抓取姿态预测网络。

### 构建数据集

​	我们在GraspNet-1Billion数据集的基础上，首先对realsense相机拍出的图像进行渲染，生成水平间距为0.055m的双目图像（这部分不是我完成的），然后在双目图像的基础上分别进行插帧，使用的方法是CVPR2023的论文中提出的EMA-VFI，可以指定两张图像中插帧的数量，随后将这批图像转化为帧率为120的视频。最后将视频转换成spike脉冲。

![data](D:\OneDrive\桌面\BaiduSyncdisk\amao996.github.io\blogs\project\spike_robot\data.png)

### 模型架构

​	下述模型为graspnet中的模型，本项目基于这个模型将输入和point encoder-decoder部分修改。

![model2](I:\amao996.github.io\blogs\project\spike_robot\model2.png)

​	具体修改框架如下图所示

![model1](I:\amao996.github.io\blogs\project\spike_robot\model1.png)

​	首先对spike stream进行DSFT转换，所谓DSFT即differential of spike firing time，提出于论文Learning Optical Flow from Continuous Spike Streams，这种方法可以对spike进行高效表示，随后进行特征提取，该部分与RAFT论文中相似。随后通过构建3D关联矩阵，将双目信息进行融合。最后通过GRU提取到特征。