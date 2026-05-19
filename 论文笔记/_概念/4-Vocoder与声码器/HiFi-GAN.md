---
type: concept
aliases: [HiFi-GAN, HifiGAN]
---

# HiFi-GAN

## 定义
Kong et al. NeurIPS 2020 提出的 GAN-based 高保真神经声码器，把 Mel spectrogram 转换为 waveform。

## 核心要点
1. Multi-Period Discriminator (MPD) + Multi-Scale Discriminator (MSD)
2. Generator 用 transposed conv + MRF blocks
3. 实时性能好，质量接近真人
4. 事实标准的开源 vocoder

## 代表工作
- [[OmniFlatten]] / [[CosyVoice]] / [[VITS]] 等几乎所有 TTS 系统都用作最终 vocoder

## 相关概念
- [[OmniFlatten]]
