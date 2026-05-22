---
title: "Seamless: Multilingual Expressive and Streaming Speech Translation"
method_name: "SeamlessM4T"
authors: [Seamless Communication Team, Loïc Barrault, Yu-An Chung, Mariano Coria Meglioli]
year: 2023
venue: arXiv
arxiv_id: null
pdf_path: "assets/papers/SeamlessM4T.pdf"
library_source: "高德文献库"
source_topic: "SSL"
tags: [classic, translation, speech-llm]
created: 2026-05-22
---

# SeamlessM4T: Multilingual Expressive Streaming Speech Translation

## 📌 一句话

Meta 发布的**多语言端到端语音翻译**系统系列（SeamlessM4T v2 / SeamlessExpressive / SeamlessStreaming），支持 ~100 种语言的 S2ST / S2TT / ASR，在 expressiveness（韵律保持）和 streaming（低延迟）两个方向都有突破。

## 🛠 核心方法

**输入 → 输出**: speech (source language) → speech/text (target language)

**架构组件**（按数据流顺序）:
1. **w2v-BERT 2.0 Encoder**: 自监督预训练的多语言语音编码器
2. **NLLB Text Decoder**: 复用 [[NLLB]] 的翻译能力做 text 生成
3. **Unit-based Vocoder (HiFi-GAN)**: 离散 speech unit → 目标语言波形
4. **Streaming via EMMA**: Efficient Monotonic Multihead Attention 实现流式翻译
5. **Expressiveness Module**: 保持源语音的韵律/情感/节奏特征到目标语音

**关键创新**: 首次在单一模型中同时实现**多语言 + 流式 + 表达力保持**三个目标的语音翻译，统一了之前分散的研究方向。

## 📊 关键结果 / 评测

- 首页未给出具体数字，待全文确认
- 支持 ~100 种语言的 S2ST / S2TT / ASR

## 💡 借鉴意义（一句话）

做语音翻译 / 多语言 TTS 的人**必读**——SeamlessM4T 是当前最完整的端到端语音翻译系统，其 w2v-BERT 2.0 + NLLB 组合被广泛借鉴。

## 🔗 链接

- PDF: [[assets/papers/SeamlessM4T.pdf|本地 PDF]]
- 源目录: `SSL/2312.05187v1.pdf`
