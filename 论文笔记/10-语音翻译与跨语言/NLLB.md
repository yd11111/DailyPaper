---
title: "No Language Left Behind: Scaling Human-Centered Machine Translation"
method_name: "NLLB"
authors: [NLLB Team, Marta R. Costa-jussà, James Cross, Onur Çelebi, Maha Elbayad]
year: 2022
venue: arXiv
arxiv_id: null
pdf_path: "assets/papers/NLLB.pdf"
library_source: "高德文献库"
source_topic: "SSL"
tags: [classic, translation]
created: 2026-05-22
---

# NLLB: No Language Left Behind

## 📌 一句话

Meta AI 发布的 **200 语种**机器翻译模型，训练数据通过大规模自动挖掘 + 质量过滤获得，覆盖大量低资源语言。虽然主要是文本翻译，但其数据管道和多语言表示被 Seamless 系列语音翻译模型继承。

## 🛠 核心方法

**输入 → 输出**: text (source language) → text (target language)，覆盖 200 种语言

**架构组件**（按数据流顺序）:
1. **Data Mining Pipeline**: 大规模互联网平行语料自动挖掘（CCMatrix / CCAligned 等）
2. **Quality Filtering**: 语种 ID + LASER 嵌入相似度 + toxicity filter
3. **Sparsely-Gated MoE Transformer**: 54B 参数的 Mixture-of-Experts 翻译模型
4. **Evaluation**: FLORES-200 多语言 benchmark

**关键创新**: 首次把高质量机器翻译扩展到 200 种语言（包括大量非洲/东南亚/太平洋岛国语言），数据管道和多语言 tokenizer 被 [[Seamless]] 等语音翻译工作复用。

## 📊 关键结果 / 评测

- 覆盖 200 种语言，FLORES-200 上 44% 的语言对 BLEU 提升超过 15 点
- 54B MoE 模型 + distilled 3.3B/1.3B 版本

## 💡 借鉴意义（一句话）

做语音翻译（S2ST）的人需了解——NLLB 的多语言 tokenizer 和训练数据是 SeamlessM4T 的文本翻译基座。

## 🔗 链接

- PDF: [[assets/papers/NLLB.pdf|本地 PDF]]
- 源目录: `SSL/2207.04672v3.pdf`
