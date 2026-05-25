---
type: concept
aliases: [TransformerTTS]
---

# Transformer TTS

## 定义

将 Transformer 架构应用于 TTS 的自回归模型，用 encoder-decoder attention 替代 Tacotron 的 RNN，实现更强的长距离依赖建模，但仍是 AR 生成，推理速度慢。

## 核心要点

1. 结构: Phoneme Encoder → Decoder (AR, 逐帧生成 mel-spectrogram)
2. 比 Tacotron 2 的 RNN 更容易并行训练
3. 推理仍是自回归，RTF ~0.93
4. FastSpeech 以 Transformer TTS 为 teacher 做蒸馏

## 评测/常见数字

- LJSpeech MOS ~3.72 (with PWG)

## 代表工作

- Neural Speech Synthesis with Transformer Network (Li et al., 2019)
- [[FastSpeech]]: 以 Transformer TTS 为 teacher

## 相关概念

- [[Tacotron 2]]
- [[FastSpeech]]
- [[Self-Attention]]
