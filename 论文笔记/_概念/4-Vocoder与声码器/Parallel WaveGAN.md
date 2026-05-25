---
type: concept
aliases: [PWG]
---

# Parallel WaveGAN

## 定义

基于 GAN 的并行波形生成声码器，使用非因果 WaveNet 作为生成器 + multi-resolution STFT loss + 对抗损失训练，实现高质量的 Mel-to-Waveform 并行合成。

## 核心要点

1. 替代 AR WaveNet，实现并行波形生成
2. 判别器: 10 层 non-causal dilated 1D Conv + Leaky ReLU
3. 损失: multi-resolution STFT loss + LSGAN adversarial loss
4. 被 HiFi-GAN 等后续工作超越

## 代表工作

- [[FastSpeech2]]: 作为 vocoder 使用
- Parallel WaveGAN (Yamamoto et al., 2020)

## 评测/常见数字

- FastSpeech 2 + PWG: MOS 3.83 (LJSpeech)

## 相关概念

- [[WaveNet]]
- [[HiFi-GAN]]
- [[Mel-Spectrogram]]
