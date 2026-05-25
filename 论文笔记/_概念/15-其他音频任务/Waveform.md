---
type: concept
aliases: [波形, 原始波形, Raw Waveform]
---

# Waveform

## 定义

音频信号的原始时域表示，由离散采样点序列组成。常见采样率: 16kHz (ASR)、22.05kHz (TTS)、24kHz / 44.1kHz (高质量音频)。是所有音频处理的基础输入/最终输出形式。

## 核心要点

1. 一维信号，每秒包含采样率个数据点（如 22050Hz → 每秒 22050 个点）
2. 可通过 STFT 转为频域表示（Mel-Spectrogram 等）
3. TTS 最终输出 waveform（通过 vocoder 或端到端生成）
4. 直接建模 waveform 比建模 mel-spectrogram 困难得多（更多 variance，如 phase）

## 代表工作

- [[WaveNet]]: 首个直接生成 waveform 的神经模型
- [[FastSpeech2]]: FastSpeech 2s 首次全并行 text-to-waveform

## 相关概念

- [[Mel-Spectrogram]]
- [[HiFi-GAN]]
- [[WaveNet]]
