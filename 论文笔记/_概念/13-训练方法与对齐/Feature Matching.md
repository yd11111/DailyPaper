---
type: concept
aliases: [Feature Matching Loss, 特征匹配损失]
---

# Feature Matching

## 定义

GAN 训练中常用的辅助 loss：让生成样本和真实样本在判别器各中间层激活上 L1 / L2 距离最小化。相比纯对抗目标更稳定，提供更细粒度的 perceptual 监督，是 HiFi-GAN 之后 audio GAN 训练的标准配方之一。

## 数学形式

$$
\mathcal{L}_{\mathrm{fm}} = \sum_{i=1}^{N} \frac{1}{T_i} \left\| D_i(x_{\mathrm{real}}) - D_i(x_{\mathrm{fake}}) \right\|_1
$$

其中 $D_i$ 是判别器第 $i$ 层激活，$T_i$ 是该层元素数。

## 核心要点

1. **稳定 GAN 训练**：单纯 adversarial loss 易模式崩塌，feature matching 给出"过程"约束。
2. **配合 MPD/MSD/[[MFD]]**：每个判别器分支都各自算 feature matching 后求和。
3. **音频 codec / vocoder 必备**：[[HiFi-GAN]] / [[DAC]] / [[Vocos]] / [[LoSATok]] 等几乎都用。

## 代表工作

- HiFi-GAN (Kong et al., 2020): 引入到音频 vocoder。
- 几乎所有现代神经 vocoder / codec / unified tokenizer。

## 相关概念

- [[MFD]] / Multi-Period Discriminator / Multi-Scale Discriminator
- [[Adversarial Loss]]
- [[Mel-Spectrogram]] 重建 loss：通常一起使用
