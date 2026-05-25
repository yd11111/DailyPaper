---
type: concept
aliases: [MFA]
---

# Montreal Forced Alignment

## 定义

开源的语音-文本强制对齐工具，基于 Kaldi，能将文本的 phoneme 序列与音频在时间上精确对齐，输出每个 phoneme 的起止时间。广泛用于 TTS 训练数据的 duration 标注。

## 核心要点

1. 输出 phoneme-level duration annotation，精度高于 AR teacher attention 提取的 duration
2. FastSpeech 2 中 MFA duration 误差比 teacher duration 低 36.6% (12.47ms vs 19.68ms)
3. 需要预训练的声学模型和发音词典
4. 后续被 MAS 等端到端方法部分替代

## 代表工作

- [[FastSpeech2]]: 用 MFA 替代 teacher attention 获取 duration

## 相关概念

- [[Forced Alignment]]
- [[Duration Predictor]]
- [[Monotonic Alignment Search]]
