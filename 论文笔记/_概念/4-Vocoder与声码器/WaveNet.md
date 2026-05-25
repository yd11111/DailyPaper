---
type: concept
aliases: [WaveNet Vocoder]
---

# WaveNet

## 定义

DeepMind 提出的自回归波形生成模型，使用 dilated causal convolution 逐采样点生成音频波形。质量极高但推理极慢，催生了后续 Parallel WaveGAN / HiFi-GAN 等并行声码器。

## 核心要点

1. 逐采样点 AR 生成，24kHz 采样率下每秒需生成 24000 个点
2. Dilated causal convolution 指数级增长感受野
3. Gated activation: $\tanh(W_f * x) \odot \sigma(W_g * x)$
4. 后续 non-causal 变体用于 Parallel WaveGAN 和 FastSpeech 2s

## 代表工作

- WaveNet (van den Oord et al., 2016)
- [[Parallel WaveGAN]]: 非因果 WaveNet + GAN
- [[FastSpeech2]]: FastSpeech 2s waveform decoder 基于 non-causal WaveNet
- [[VITS]]: Posterior Encoder 和 Flow coupling layer 使用 WaveNet 残差块

## 相关概念

- [[HiFi-GAN]]
- [[Parallel WaveGAN]]
- [[Convolution]]
