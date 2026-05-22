---
title: "XTTS: a Massively Multilingual Zero-Shot Text-to-Speech Model"
method_name: "XTTS"
authors: [Edresson Casanova, Kelly Davis, Eren Gölge, Görkem Göknar, Iulian Gulea]
year: 2024
venue: arXiv
arxiv_id: "2406.04904"
pdf_path: "assets/papers/XTTS.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, tts]
created: 2026-05-22
---

# XTTS: Massively Multilingual Zero-Shot TTS

## 📌 一句话

Coqui 推出的**大规模多语言 zero-shot TTS**——在 [[YourTTS]] / [[Tortoise]] 基础上发展而来，VQ-VAE + GPT-2 + HiFi-GAN 架构，支持 17 种语言的 zero-shot voice cloning，完全开源。

## 🛠 核心方法

**输入 → 输出**: text + 6s reference audio → waveform

**架构组件**（按数据流顺序）:
1. **VQ-VAE**: 语音 → 离散 token
2. **GPT-2 Encoder + Conditioning Encoder**: text + speaker reference → VQ token 序列
3. **HiFi-GAN Decoder**: VQ token → waveform

**关键创新**: 将 Tortoise 的架构**扩展到 17 种语言**——通过大规模多语言数据训练，实现跨语言 zero-shot voice cloning，且模型完全开源。

## 🖼 架构图

![Figure 1: XTTS 训练架构总览——VQ-VAE + GPT-2 + HiFi-GAN](https://ar5iv.labs.arxiv.org/html/2406.04904/assets/Images/XTTS.png)

## 📊 关键结果 / 评测

- 英文 CER: 0.543%（优于 HierSpeech++ 0.774%、Tortoise 1.093%）
- 16 语言平均 CER: 2.06%（YourTTS 4.79%）
- 16 语言平均 speaker similarity (SECS): 0.505（YourTTS 0.470）
- 主观评测: CMOS +0.41 vs HierSpeech++, +0.92 vs Mega-TTS 2

## 💡 借鉴意义（一句话）

做多语言 TTS 的人参考——XTTS 是目前覆盖语言最多的开源 zero-shot TTS 之一。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2406.04904
- PDF: [[assets/papers/XTTS.pdf|本地 PDF]]
- 源目录: `TTS-LLM/XTTS.pdf`
