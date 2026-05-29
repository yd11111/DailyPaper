---
type: concept
aliases: [Multi-Frequency Discriminator, 多频判别器]
---

# MFD (Multi-Frequency Discriminator)

## 定义

一类用于神经声码器 / audio codec 对抗训练的多频段判别器。把音频分解到不同频率分量后，对每段单独输入卷积判别网络做真假判别，相比单一判别器（如 MPD / MSD）能更好捕捉细粒度频谱差异。由 [[Llasa]] (Ye et al., 2025) 等工作中使用，并被 [[LoSATok]] 用作其 GAN 训练的判别器。

## 核心要点

1. **多频段输入**：通过 STFT 或 mel 子带划分后，对每段分别判别。
2. **配合 hinge loss + feature matching**：典型 GAN 训练配方，feature matching 提供 perceptual 监督。
3. **替代/补充 MPD/MSD**：HiFi-GAN 时代用 multi-period (MPD) + multi-scale (MSD) discriminator；MFD 在频域而非时域分解。
4. **适合 low-bitrate codec**：低码率时频谱细节易丢失，MFD 强调频谱保真度。

## 代表工作

- [[Llasa]] (Ye et al., 2025): 引入 MFD 用于 LLM-based speech synthesis 中的 codec 训练。
- [[LoSATok]]: 复用作其 adversarial loss 中的判别器。

## 相关概念

- [[Feature Matching]]：通常与 MFD 联合使用的 perceptual loss。
- [[Adversarial Loss]] / [[Hinge Loss]]：MFD 的对抗目标。
- [[HiFi-GAN]] / Multi-Period Discriminator / Multi-Scale Discriminator：先前的判别器范式。
- [[Vocos]]：常配合此类判别器的声码器。
