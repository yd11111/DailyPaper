---
type: concept
aliases: []
---

# SpeechTokenizer

## 定义

复旦 2023 年提出的统一语音 tokenizer，把语义/声学信息**分层级**编码到 RVQ 码本：第 1 层尽量承载语义（用 HuBERT 蒸馏对齐），后续层补声学细节。专为 speech LLM 设计。

## 核心要点

1. 第 1 码本对齐 [[HuBERT]] 语义；后续层学声学残差
2. 4 码本 / 300 token/s
3. 在低层码本上具有"semantic ↔ acoustic"分离的便利性

## 代表工作

- 原论文 (arXiv 2308.16692, Zhang et al. 2023)
- [[VibeVoice]] tokenizer 重建对比基线

## 评测/常见数字

- LibriTTS test-clean PESQ: 1.931 / UTMOS: 3.563
- LibriTTS test-other PESQ: 1.737 / UTMOS: 3.018

## 相关概念

- [[RVQ]]
- [[EnCodec]]
- [[DAC]]
