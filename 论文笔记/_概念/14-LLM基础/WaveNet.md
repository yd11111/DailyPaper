---
type: concept
aliases: [WaveNet]
---

# WaveNet

## 定义
DeepMind 提出的自回归神经声码器，直接在原始波形采样点级别进行建模，使用因果膨胀卷积（causal dilated convolution）逐样本生成音频。TTS 声码器领域的开创性工作。

## 核心要点
1. 逐样本自回归生成，质量极高但推理极慢（16kHz 下每秒需生成 16000 个采样点）
2. 使用因果膨胀卷积堆叠扩大感受野，同时保持因果性
3. 后续催生了 Parallel WaveNet、WaveGlow、WaveRNN 等加速方案
4. 进一步推动了 HiFi-GAN、BigVGAN 等 GAN-based 声码器的发展

## 代表工作
- van den Oord et al. 2016: WaveNet 原始论文
- Parallel WaveNet (2017): 知识蒸馏加速版
- [[WaveGlow]]: Flow-based 并行声码器

## 相关概念
- [[WaveGlow]]
- [[HiFi-GAN]]
- [[Mel-Spectrogram]]
