---
type: concept
aliases: [卷积, Conv1D, 1D Convolution, Dilated Convolution]
---

# Convolution

## 定义

卷积运算，在深度学习中用于提取局部特征模式。1D Convolution 在序列建模（语音、文本）中广泛使用，Dilated Convolution 通过膨胀率扩大感受野。

## 核心要点

1. 1D Conv 在 TTS 中常用于 phoneme/frame 级别的局部特征提取
2. Dilated Conv 在 WaveNet / Vocoder 中用于高效建模长距离依赖
3. 常与 Self-Attention 互补使用（如 FFT block = Self-Attention + Conv1D）

## 代表工作

- [[FastSpeech2]]: FFT block 中的 1D Conv (kernel=9)
- [[WaveNet]]: Dilated causal convolution
- [[HiFi-GAN]]: Multi-Receptive Field Fusion

## 相关概念

- [[Self-Attention]]
- [[Feed-Forward Transformer]]
