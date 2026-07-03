---
type: concept
aliases: [HiFTNet Vocoder, HiFi-Transformer-Net]
---

# HiFTNet

## 定义
HiFTNet 是一种高保真语音声码器（vocoder），将 mel-spectrogram 转换为波形。被 [[CosyVoice2]] 等系统采用作为最终的波形合成模块。

## 核心要点
1. 将 Transformer 与 HiFi-GAN 风格的生成器结合
2. 被 CosyVoice 系列系统用作默认声码器
3. 在 Flow Matching TTS 管线中位于最末端：语义 token → FM 解码器 → mel → HiFTNet → waveform

## 代表工作
- [[CosyVoice2]]: 使用 HiFTNet 作为声码器
- [[UnifiedGuidanceFM]]: TTS 评测中使用 HiFTNet vocoder

## 相关概念
- [[HiFi-GAN]]
- [[Voice Conversion]]
