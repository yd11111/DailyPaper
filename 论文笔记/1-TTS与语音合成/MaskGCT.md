---
title: "MaskGCT: Zero-Shot Text-to-Speech with Masked Generative Codec Transformer"
method_name: "MaskGCT"
authors: [Yuancheng Wang, Haoyue Zhan, Liwei Liu, Ruihong Zeng, Haotian Guo]
year: 2024
venue: arXiv
arxiv_id: "2409.00750"
pdf_path: "assets/papers/MaskGCT.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, tts]
created: 2026-05-22
---

# MaskGCT: Masked Generative Codec Transformer for Zero-Shot TTS

## 📌 一句话

港中深提出的 **non-autoregressive** zero-shot TTS——用 masked generative modeling（类 MaskGIT）替代 AR 和 diffusion，两阶段 text→semantic token + semantic→acoustic token，推理速度快且无需时长对齐。

## 🛠 核心方法

**输入 → 输出**: text + speech prompt → semantic tokens → acoustic tokens → waveform

**架构组件**（按数据流顺序）:
1. **Speech Semantic Codec**: 从 [[w2v-BERT]] / [[HuBERT]] 提取语义 token
2. **T2S (Text-to-Semantic)**: masked prediction，text + prompt semantic → target semantic tokens
3. **S2A (Semantic-to-Acoustic)**: masked prediction，semantic → multi-layer acoustic tokens
4. **Speech Acoustic Codec**: acoustic tokens → waveform

**关键创新**: 完全基于 **masked generative transformer**（非 AR 非 diffusion），text-to-semantic 阶段不需要 phoneme 对齐或时长预测，模型自动学习时长；推理时通过迭代 unmask 实现并行生成。

## 🖼 架构图

![Figure 1: MaskGCT 系统总览——semantic codec + T2S + S2A + acoustic codec](https://ar5iv.labs.arxiv.org/html/2409.00750/assets/x1.png)

## 📊 关键结果 / 评测

- LibriSpeech test-clean: WER 2.63%, SIM-O 0.688
- Seed-TTS-eval en: WER 2.00%, SIM-O 0.760
- 推理速度: 比 AR 方法快 2-5x

## 💡 借鉴意义（一句话）

做 TTS 的人关注——MaskGCT 证明了 masked generative modeling 可以替代 AR/diffusion 做高质量 zero-shot TTS，且无需显式时长建模。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2409.00750
- PDF: [[assets/papers/MaskGCT.pdf|本地 PDF]]
- 源目录: `TTS-LLM/MaskGCT.pdf`
