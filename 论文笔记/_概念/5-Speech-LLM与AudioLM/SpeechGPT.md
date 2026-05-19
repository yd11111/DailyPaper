---
type: concept
aliases: [SpeechGPT]
---

# SpeechGPT

## 定义
复旦 2023 年提出的早期 speech-text 多模态 LLM，把语音离散化后直接作为 LLM 词表的扩展。

## 核心要点
1. 三阶段训练：modality adaptation → cross-modal instruction → chain-of-modality
2. Discrete unit 来自 [[HuBERT]] k-means
3. 首次把 speech 当 "另一种语言" 让 LLM 学

## 代表工作
- [[OmniFlatten]] 与之类似都用离散 token，但 SpeechGPT 是半双工

## 相关概念
- [[OmniFlatten]]
