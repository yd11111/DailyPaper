---
type: concept
aliases: [AudioFlamingo, Audio Flamingo]
---

# Audio-Flamingo

## 定义

NVIDIA 2024 年提出的音频大语言模型 (Kong et al., ICML 2024)。把 [[Flamingo]] 视觉-语言架构搬到音频域：用 CLAP-style 音频编码器 + few-shot in-context learning + dialogue 能力，能在 audio captioning、AQA、AudioSet 分类等任务上做 zero-shot / few-shot。

## 核心要点

1. **跨注意力 + 门控**：音频特征通过 cross-attention 注入冻结 LLM，类似原 Flamingo
2. **few-shot 能力**：支持用户在 prompt 里给几个音频-文本对作示例
3. **dialogue 模式**：可多轮对话谈论同一段音频
4. **后续 v2**：扩展上下文长度 + 更多任务
5. **不输出音频**：典型 LALM 局限——只听 / 只说文本

## 代表工作

- 原论文 (arXiv 2402.01831)
- Audio-Flamingo 2 (2024 后续)
- [[MUSA]]：作为 LALM 选择性听觉评测的主要被测模型之一

## 评测/常见数字

- AudioCaps captioning: CIDEr 接近或略超 Whisper-LLM cascade
- AQA few-shot 准确率显著高于纯 zero-shot baseline

## 相关概念

- [[LALM]]
- [[Flamingo]]
- [[CLAP]]
- [[SALMONN]]
