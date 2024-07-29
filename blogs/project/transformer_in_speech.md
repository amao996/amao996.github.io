本项目为基于经典Transformer架构(源于Attention is all you need论文)实现语音识别并对模型进行适当改进。

### 数据集

本项目采用的数据集是[Aishell1](https://www.openslr.org/33/)

### 语音信号的特征提取

常用的特征提取方法包括梅尔频谱图（Mel-spectrogram）、梅尔倒谱系数（MFCCs）、滤波器组特征（Filter Bank Features, fbank）等。

本项目使用的是梅尔频谱图法，具体如下：

```python
def extract_feature(input_file, feature='fbank', dim=80, cmvn=True, delta=False, delta_delta=False,
                    window_size=25, stride=10, save_feature=None):
    y, sr = librosa.load(input_file, sr=sample_rate)
    yt, _ = librosa.effects.trim(y, top_db=20)  # 去除静音部分
    yt = normalize(yt)
    ws = int(sr * 0.001 * window_size)
    st = int(sr * 0.001 * stride)
    feat = librosa.feature.melspectrogram(y=yt, sr=sr, n_mels=dim,
                                          n_fft=ws, hop_length=st)
    feat = np.log(feat + 1e-6)
    feat = [feat]
    if delta:
        feat.append(librosa.feature.delta(feat[0]))

    if delta_delta:
        feat.append(librosa.feature.delta(feat[0], order=2))
    feat = np.concatenate(feat, axis=0)
    
    if cmvn:
        feat = (feat - feat.mean(axis=1)[:, np.newaxis]) / (feat.std(axis=1) + 1e-16)[:, np.newaxis]
    if save_feature is not None:
        tmp = np.swapaxes(feat, 0, 1).astype('float32')
        np.save(save_feature, tmp)
        return len(tmp)
    else:
        return np.swapaxes(feat, 0, 1).astype('float32')
```

首先加载音频，去除静音部分并归一化音频信号，随后计算梅尔频谱图并进行对数变换，然后计算差分特征（可选项），最后应用倒谱均值方差归一化得到最值特征。短时傅里叶变换是计算梅尔频谱图的基础，具体计算在librosa.feature.melspectrogram函数内部实现。

### BaseModel

首先复现了speechTransformer作为BaseModel

### 模型改进

