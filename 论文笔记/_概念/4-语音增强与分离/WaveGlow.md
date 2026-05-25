---
type: concept
aliases: [WaveGlow]
---

# WaveGlow

## 定义
NVIDIA 提出的基于 Flow 的并行声码器，将 Glow 的 flow-based 生成模型应用于语音波形生成，实现 Mel-Spectrogram 到 waveform 的并行合成。

## 核心要点
1. 基于 normalizing flow，训练时最大化似然，推理时从高斯噪声反向采样
2. 无需自回归逐样本生成，推理速度远快于 WaveNet
3. 是 FastSpeech 等早期 NAR TTS 系统的默认声码器
4. 后被 HiFi-GAN、BigVGAN 等 GAN-based 声码器超越（更快更好）

## 代表工作
- Prenger et al. ICASSP 2019: WaveGlow 原始论文
- [[FastSpeech]]: 使用 WaveGlow 作为声码器

## 相关概念
- [[Mel-Spectrogram]]
- [[WaveNet]]
- [[HiFi-GAN]]
