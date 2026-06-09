---
type: concept
aliases: [Nano-Codec]
---

# NanoCodec

## 定义
NanoCodec 是一个低码率神经音频编解码器，使用 [[FSQ]]（Finite Scalar Quantization）而非传统 [[RVQ]]，实现极低码率（0.6 kbps）的语音编码。

## 核心要点
1. 码率 0.6 kbps，帧率 12.5 Hz
2. 使用 FSQ 量化，4 个码道，每道词表大小 4037
3. 设计目标是为 Speech LLM 提供紧凑的离散语音表示

## 评测/常见数字
- 码率：0.6 kbps（极低）
- 帧率：12.5 Hz
- 码道数：4
- 词表大小：4037/道

## 代表工作
- [[IRAF]]: 作为语音 token 提取器

## 相关概念
- [[FSQ]]
- [[Audio Codec]]
- [[EnCodec]]
- [[Mimi]]
