本项目为基于经典Transformer架构(源于Attention is all you need论文)实现语音识别并对模型进行适当改进。

### 数据集

本项目采用的数据集是[Aishell1](https://www.openslr.org/33/)

### 语音信号的特征提取

常用的特征提取方法包括梅尔频谱图（Mel-spectrogram）、梅尔倒谱系数（MFCCs）、滤波器组特征（Filter Bank Features, fbank）等。

本项目使用的是梅尔频谱图法，具体如下：首先加载音频，去除静音部分并归一化音频信号，随后计算梅尔频谱图并进行对数变换，然后计算差分特征（可选项），最后应用倒谱均值方差归一化得到最值特征。短时傅里叶变换是计算梅尔频谱图的基础，具体计算在librosa.feature.melspectrogram函数内部实现。

```python
def extract_feature(input_file, feature='fbank', dim=80, cmvn=True, delta=False, delta_delta=False,
                    window_size=25, stride=10, save_feature=None):
    y, sr = librosa.load(input_file, sr=sample_rate)
    yt, _ = librosa.effects.trim(y, top_db=20)  # 去除静音部分
    yt = normalize(yt)	# [-1, 1]
    ws = int(sr * 0.001 * window_size)
    st = int(sr * 0.001 * stride)
    # 提取特征
    feat = librosa.feature.melspectrogram(y=yt, sr=sr, n_mels=dim,
                                          n_fft=ws, hop_length=st)
    feat = np.log(feat + 1e-6)
    # 计算差分特征
    feat = [feat]
    if delta:
        feat.append(librosa.feature.delta(feat[0]))

    if delta_delta:
        feat.append(librosa.feature.delta(feat[0], order=2))
    feat = np.concatenate(feat, axis=0)
    # 倒谱均值方差归一化
    if cmvn:
        feat = (feat - feat.mean(axis=1)[:, np.newaxis]) / (feat.std(axis=1) + 1e-16)[:, np.newaxis]
    if save_feature is not None:
        tmp = np.swapaxes(feat, 0, 1).astype('float32')
        np.save(save_feature, tmp)
        return len(tmp)
    else:
        return np.swapaxes(feat, 0, 1).astype('float32')
```

- [梅尔频谱图法](https://blog.csdn.net/qq_44250700/article/details/125372510)：首先对音频信号进行短时傅里叶变换（STFT），随后将频谱图转换为梅尔尺度，最后计算幅值谱并取对数。
- 梅尔倒谱系数：首先计算梅尔频谱图，然后对梅尔频谱图取对数，最后进行离散余弦变换（DCT）保留前一部分系数。
- 差分特征：通过对时间序列特征进行一阶和二阶差分，用于捕捉特征随时间变化的趋势。
- 倒谱均值方差归一化：$$ C M V N ( f ) = \frac { f - \mu _ { f } } { \sigma _ { f } }$$，其中$$\mu _ { f }$$为特征的均值，$$\sigma _ { f }$$为特征的标准差。目的是使得特征在每个维度上具有0均值和单位方差，以减少不同录音条件对特征的影响。

### BaseModel

首先复现了speechTransformer作为BaseModel

### 评价指标（CER）

使用字符错误率作为该模型的评价指标，计算预测序列和真实序列之间的Levenshtein距离（两个字符串之间的最小编辑操作次数，包括插入、删除和替换，以将一个字符串转换为另一个字符串），通过动态规划实现该过程如下：

定义dp：$$dp[i][j]$$表示将字符串u的前i个字符转换为字符串v的前j个字符所需的最小编辑操作数；

状态转移方程：

如何已经计算好$$dp[i-1][j-1]$$，如果$$u[i-1]=v[j-1]$$，则不需要额外编辑操作，如果$$u[i-1] \neq v[j-1]$$，则可以进行替换操作（$$dp[i-1][j-1] + 1$$）、删除操作（$$dp[i-1][j] + 1$$）、插入操作（$$dp[i][j-1] + 1$$），取三者的最小值。
$$
dp[i][j] = \left\{
\begin{array}{ll}
dp[i-1][j-1] & \text{if } u[i-1]=v[j-1] \\
1 + min(dp[i-1][j-1],dp[i-1][j],dp[i][j-1]) & \text{if } u[i-1] \neq v[j-1]
\end{array}
\right.
$$


初始化：$$dp[0][0] = 0, dp[i][0] = i, dp[0][j] = j$$；

最终得到插入、删除、替换字符的数目分别是i、d、s，用三者之和除以真实序列的长度得到字符错误率CER。

### 模型改进

low frame rate

focal loss

conformer

spec augment