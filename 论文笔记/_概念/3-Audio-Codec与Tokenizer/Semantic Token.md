---
type: concept
aliases: [Semantic Token, 语义 Token]
---

# Semantic Token

## 定义
承载语言/内容信息的语音离散 token，通常来自 SSL 模型（如 [[HuBERT]]、[[WavLM]]）做 k-means 聚类，或用多语种 ASR 监督训练得到。

## 核心要点
1. 比 acoustic token 更接近 phoneme 层级
2. 适合 LLM 做语言建模，但不能直接重建高质量音频
3. 需配合 acoustic decoder（如 [[OT-CFM]] + Vocoder）回到 waveform

## 代表工作
- [[AudioLM]]: 首次提出 semantic token + acoustic token 分层建模，用 w2v-BERT 第 7 层 k-means 聚类（K=1024, 250 bps）
- [[CosyVoice]]、[[OmniFlatten]] 都用 semantic token 而非 acoustic token

## 相关概念
- [[OmniFlatten]]
