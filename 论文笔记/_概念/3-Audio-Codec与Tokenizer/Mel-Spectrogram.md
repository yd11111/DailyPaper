---
type: concept
aliases: [Mel频谱图, 梅尔频谱, Mel Spectrogram, Mel-Spec]
---

# Mel-Spectrogram

## 定义

对数 Mel 频谱图，将音频波形通过 STFT + Mel 滤波器组转化为二维时频表示。是 TTS（声码器输入）、ASR（特征提取）等任务中最常用的中间表示之一。

## 数学形式

$$
\text{Mel}(f, t) = \log\left(\mathbf{M} \cdot |\text{STFT}(x)|^2 + \epsilon\right)
$$

- $\mathbf{M} \in \mathbb{R}^{N_{mel} \times N_{fft}/2+1}$: Mel 滤波器组矩阵
- 常见维度：80 或 100 个 Mel bin
- 常见帧移（hop）：10ms / 12.5ms

## 核心要点

1. 相比原始波形，Mel 频谱图大幅降低维度，保留人耳感知相关的频率信息
2. 是大多数声码器（HiFi-GAN、BigVGAN、Vocos）的输入格式
3. 在级联 TTS 系统中通常是中间表示：文本 → Mel → Waveform

## 代表工作

- [[IndexTTS2]]: S2M 模块输出 Mel-Spectrogram，由 BigVGAN 转为波形

## 相关概念

- [[BigVGAN]]
- [[Flow Matching]]
