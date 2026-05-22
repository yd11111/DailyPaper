---
title: "Fish-Speech: Leveraging Large Language Models for Advanced Multilingual Text-to-Speech Synthesis"
method_name: "Fish-Speech"
authors: [Shijia Liao, Yuxuan Wang, Tianyu Li, Yifan Cheng, Ruoyi Zhang]
year: 2024
venue: arXiv
arxiv_id: "2411.01156"
pdf_path: "assets/papers/Fish-Speech.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, tts]
created: 2026-05-22
---

# Fish-Speech: LLM-based Multilingual TTS

## 📌 一句话

Fish Audio 推出的开源 TTS——**Dual-AR** 架构（slow AR 建模语义 + fast AR 建模声学细节）+ FireFly GAN vocoder，完全开源，支持多语言 zero-shot voice cloning。

## 🛠 核心方法

**输入 → 输出**: text + reference audio → speech tokens → waveform

**架构组件**（按数据流顺序）:
1. **GroupedVQ Tokenizer**: 多组 VQ 量化语音
2. **Dual-AR Model**: slow AR（粗粒度 token 序列）+ fast AR（细粒度 token 补充）
3. **FireFly GAN**: token → waveform 的高效 vocoder

**关键创新**: **Dual-AR**（双速率自回归）——slow AR 降低了序列长度（更快推理），fast AR 补充声学细节（不损失质量），兼顾速度和质量。

## 🖼 架构图

![Figure 1: Fish Speech 架构](https://ar5iv.labs.arxiv.org/html/2411.01156/assets/FS_architecture_2.0.png)

![Figure 2: Dual-AR 框架详解](https://ar5iv.labs.arxiv.org/html/2411.01156/assets/DualAR2_0.png)

## 📊 关键结果 / 评测

- 完全开源（模型 + 代码 + 训练）
- 支持中/英/日等多语言
- 实时率优于同质量的 AR-only 方案

## 💡 借鉴意义（一句话）

做开源 TTS 的人关注——Fish-Speech 的 Dual-AR 是 AR TTS 加速的一种有效方案，完全开源可直接复用。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2411.01156
- PDF: [[assets/papers/Fish-Speech.pdf|本地 PDF]]
- 源目录: `TTS-LLM/Fish-TTS.pdf`
