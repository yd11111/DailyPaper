---
type: concept
aliases: [Speech Tokenizer, 语音 Tokenizer]
---

# Speech Tokenizer

## 定义
把连续 waveform 离散化成 token 序列的模型，让语音可被 LLM 当成 "另一种语言" 处理。

## 核心要点
1. 两大流派：**语义 token**（HuBERT/CosyVoice，承载内容）vs **声学 token**（EnCodec/DAC，承载声学细节）
2. 码率 (bps)、码本数 (1 vs RVQ 多码本) 是关键设计参数
3. 单码本（如 [[CosyVoice]]、[[WavTokenizer]]）让 LLM 端处理简单

## 代表工作
- [[OmniFlatten]] 用 [[CosyVoice]] 单码本 4096 codes 的语义 tokenizer

## 相关概念
- [[OmniFlatten]]
