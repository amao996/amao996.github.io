## Advances in spike vision

### Abstract

传统视频的帧率只有几十Hz，不能记录光的高速变化过程，成为限制机器视觉速度的天花板，其根本原因在于视频概念脱胎于胶片成像，未能发挥电子和数字技术的潜力。脉冲视觉模型通过感光器件捕获光子，累积能量达到约定阈值时产生脉冲，形成脉冲的时间越长，表明收到的光信号越弱，反之光信号越强，据此可估计任意时刻的光强，从而实现连续成像。

### 脉冲影像重建

#### 脉冲间隔法(texture from interval, TFI)

利用脉冲间隔随着光强增加而减小这一特征，反映的是极短时间内的瞬时光强。

#### 时间窗平均法(texture from window, TFW)

反映的是相对更长时段内的平均光照强度，对于静态场景产生的图像通常更稳定。