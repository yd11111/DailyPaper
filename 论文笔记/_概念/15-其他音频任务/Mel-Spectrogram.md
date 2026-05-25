---
type: concept
aliases: [Mel 频谱图, Mel Spectrogram, Mel-Spec, 梅尔频谱]
---

# Mel-Spectrogram

## 定义
对数 Mel 频谱图，将音频 STFT 结果映射到 Mel 频率刻度上并取对数。是 TTS 和 ASR 中最常用的中间声学表示，通常 80 或 100 维，10-12.5 ms hop。

## 核心要点
1. 比原始波形低维，保留人耳敏感的频率信息
2. TTS 二阶段架构中：Text → Mel → Waveform（Vocoder）
3. CosyVoice 中 Flow Matching 的生成目标就是 Mel 频谱

## 代表工作
- [[CosyVoice]]: OT-CFM 生成 Mel 频谱
- [[HiFi-GAN]]: Mel → Waveform 的声码器

## 相关概念
- [[HiFi-GAN]]
