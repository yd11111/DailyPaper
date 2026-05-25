---
type: concept
aliases: [Vocoder, 声码器, Neural Vocoder]
---

# Vocoder

## 定义
声码器，将中间声学表示（通常是 [[Mel-Spectrogram]]）转换为时域波形的模型。神经声码器是现代 TTS 系统的关键组件。

## 核心要点
1. 传统两阶段 TTS: 声学模型产生 mel → vocoder 产生波形
2. 端到端系统（如 [[VITS]]）将 vocoder 集成到统一框架中
3. 主要范式：自回归（WaveNet）、GAN（[[HiFi-GAN]]、[[BigVGAN]]）、Diffusion（DiffWave）、iSTFT（[[Vocos]]）

## 代表工作
- [[WaveNet]]: 首个神经声码器，自回归，质量高但慢
- [[HiFi-GAN]]: GAN-based，速度与质量平衡的标杆
- [[Vocos]]: iSTFT-based，更轻量
- [[VITS]]: 将 vocoder 融入端到端框架

## 相关概念
- [[Mel-Spectrogram]]
- [[HiFi-GAN]]
- [[STFT]]
