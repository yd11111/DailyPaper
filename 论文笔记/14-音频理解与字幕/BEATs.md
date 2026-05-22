---
title: "BEATs: Audio Pre-Training with Acoustic Tokenizers"
method_name: "BEATs"
authors: []
year: 2022
venue: ICML 2023
arxiv_id: "2212.09058"
pdf_path: "assets/papers/BEATs.pdf"
library_source: "高德文献库"
source_topic: "tools"
tags: [classic, ssl, audio-understanding]
created: 2026-05-22
---

# BEATs: Audio Pre-Training with Acoustic Tokenizers

## 📌 一句话

微软提出的**迭代式音频预训练**框架——交替训练 acoustic tokenizer（将音频离散化为 label）和 audio SSL model（预测这些 label），每轮迭代 tokenizer 和 model 互相提升，在 AudioSet 上达到 SOTA。

## 🛠 核心方法

**输入 → 输出**: audio → audio event tags / audio understanding representations

**架构组件**（按迭代流程）:
1. **Acoustic Tokenizer**: 把音频离散化为 pseudo-label（类似 HuBERT 的 k-means，但专门为音频事件设计）
2. **Audio SSL Model (ViT)**: masked prediction 预测 tokenizer 产出的 pseudo-label
3. **Iterative Training**: tokenizer 和 SSL model 交替训练，每轮用更好的 SSL model 重新训练 tokenizer
4. **Fine-tuning**: 在 AudioSet / ESC-50 等下游任务微调

**关键创新**: 把 HuBERT 的"k-means → masked prediction"迭代思路扩展到**通用音频**领域（不只语音），且 tokenizer 不再是简单 k-means 而是可学习的网络。

## 🖼 架构图

![Figure 3: Overview of audio SSL model pre-training and fine-tuning](https://ar5iv.labs.arxiv.org/html/2212.09058/assets/Arc.png)

## 📊 关键结果 / 评测

- AudioSet: mAP 50.6%（SOTA at publication）
- ESC-50: 98.1% accuracy

## 💡 借鉴意义（一句话）

做 Audio Understanding / 音频事件检测的人**必读**——BEATs 证明了迭代式 tokenizer + SSL 在通用音频理解上的有效性。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2212.09058
- PDF: [[assets/papers/BEATs.pdf|本地 PDF]]
- 源目录: `tools/Beats.pdf`
