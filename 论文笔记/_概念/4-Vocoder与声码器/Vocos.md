---
type: concept
aliases: [vocos]
---

# Vocos

## 定义
一种基于傅里叶变换的 GAN 声码器，直接在频域生成波形。与传统时域声码器（如 [[HiFi-GAN]]）不同，Vocos 在频率域操作，使用 ConvNeXt backbone 和 iSTFT 输出层，速度更快且参数更少。

## 数学形式

$$
\hat{x} = \text{iSTFT}(\hat{A} \cdot e^{j\hat{\phi}})
$$

- 输入: Mel spectrogram 或 discrete token embedding
- 中间表示: 预测幅度 $\hat{A}$ 和相位 $\hat{\phi}$
- 输出: 波形 $\hat{x}$

## 核心要点
1. 频域操作避免了时域上采样的 artifacts，推理速度快
2. 使用 ConvNeXt block 作为 backbone，参数效率高
3. 训练使用 Multi-Period Discriminator (MPD) + Multi-Resolution Discriminator (MRD)
4. 被 GLM-TTS 扩展为 Vocos2D（用 2D 卷积替换 1D，移除 MPD，加入 Discriminator Augmentation）

## 代表工作
- Vocos (原始): 频域 GAN 声码器
- [[GLM-TTS]]: 提出 Vocos2D，用 2D 卷积 + DiT 风格残差改进 Vocos，MOS 从 3.58 提升至 4.16

## 评测/常见数字
- 推理速度: 比 [[HiFi-GAN]] 快
- 参数量: 比传统声码器少
- Vocos2D (GLM-TTS): MOS 4.16, NISQA 3.40, UTMOS 1.91

## 相关概念
- [[HiFi-GAN]]
- [[UTMOS]]
- [[MOS]]
