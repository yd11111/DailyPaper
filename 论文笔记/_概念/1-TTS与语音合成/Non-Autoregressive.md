---
type: concept
aliases: [Non-Autoregressive, NAR, 非自回归, NAR TTS, Parallel Generation]
---

# Non-Autoregressive

## 定义
一类序列生成范式，不依赖前一个 token/帧来生成当前 token/帧，而是一次性并行生成整个输出序列。在 TTS 中，NAR 模型（如 FastSpeech）并行生成所有 mel-spectrogram 帧，速度远快于 AR 模型。

## 核心要点
1. AR 模型逐帧生成，推理时间与序列长度线性相关；NAR 模型并行生成，推理时间近似常数
2. NAR TTS 的关键挑战是如何确定输出序列长度 -- FastSpeech 通过 Duration Predictor 解决
3. NAR 生成技术最早在机器翻译中提出（Gu et al. 2017），FastSpeech 将其引入 TTS
4. 后续发展：FastSpeech 2、VITS、NaturalSpeech 系列均为 NAR 或 semi-NAR

## 代表工作
- [[FastSpeech]]: 首个完全 NAR 的 mel 生成 TTS
- [[FastSpeech 2]]: 改进版 NAR TTS
- [[VITS]]: 端到端 NAR TTS

## 相关概念
- [[Autoregressive Model]]
- [[Duration Predictor]]
- [[Length Regulator]]
- [[Knowledge Distillation]]
