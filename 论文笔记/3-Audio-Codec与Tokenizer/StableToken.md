---
title: "StableToken: A Noise-Robust Semantic Speech Tokenizer for Resilient Speech LLMs"
method_name: "StableToken"
authors: []
year: 2026
venue: ICLR 2026 submission
arxiv_id: null
pdf_path: "assets/papers/StableToken.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, codec]
created: 2026-05-22
---

# StableToken: Noise-Robust Semantic Speech Tokenizer

## 📌 一句话

提出**噪声鲁棒**的语义 speech tokenizer——发现现有 semantic tokenizer（[[HuBERT]] k-means 等）对噪声非常脆弱（加一点噪声 token 序列完全变了），通过噪声感知训练和鲁棒量化设计让 token 在噪声条件下保持稳定。

## 🛠 核心方法

**输入 → 输出**: noisy speech → robust semantic tokens

**架构组件**:
1. **Noise-aware Encoder**: 编码时考虑噪声条件
2. **Robust Quantizer**: 设计噪声不变的量化目标
3. **Resilient Speech LLM**: 基于 StableToken 的下游 Speech LLM

**关键创新**: 首次系统性揭示了 semantic speech tokenizer 的**噪声脆弱性**问题，并提出针对性的鲁棒化方案。

## 📊 关键结果 / 评测

- ICLR 2026 submission（anonymous）
- 噪声条件下 token 一致性显著优于 HuBERT k-means
- 下游 ASR/TTS 在噪声场景下性能提升

## 💡 借鉴意义（一句话）

做 Speech Tokenizer / Speech LLM 的人关注——StableToken 指出了一个被忽视的问题：semantic tokenizer 的噪声鲁棒性直接影响 Speech LLM 的实际部署效果。

## 🔗 链接

- PDF: [[assets/papers/StableToken.pdf|本地 PDF]]
- 源目录: `TTS-LLM/STABLETOKEN.pdf`
