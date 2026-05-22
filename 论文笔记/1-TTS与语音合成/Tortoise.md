---
title: "Better speech synthesis through scaling"
method_name: "Tortoise"
authors: [James Betker]
year: 2023
venue: arXiv
arxiv_id: "2305.07243"
pdf_path: "assets/papers/Tortoise.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, tts]
created: 2026-05-22
---

# Tortoise: Better Speech Synthesis Through Scaling

## 📌 一句话

开源社区影响力极大的 TTS 系统——用 **AR transformer + DDPM + CLVP** 三阶段架构，借鉴 DALL-E 的图像生成范式做语音，证明了"暴力 scaling"（更大模型 + 更多数据）在 TTS 中同样有效。后来作者加入 OpenAI。

## 🛠 核心方法

**输入 → 输出**: text + reference audio → speech

**架构组件**（按数据流顺序）:
1. **VQ-VAE**: 语音 → 离散 token（MEL-based）
2. **AR Transformer**: text + reference → VQ token 序列（生成多个候选）
3. **CLVP (Contrastive Language-Voice Pretraining)**: 对 AR 候选排序，选最匹配的
4. **DDPM Decoder**: VQ token → mel spectrogram → waveform

**关键创新**: 将图像生成的 "AR + reranking + diffusion refinement" 范式完整搬到语音领域，CLVP 做候选筛选是独特设计。

## 🖼 架构图

![Figure 1: Tortoise-v2 架构设计——text + reference → AR → CLVP rerank → DDPM → waveform](https://ar5iv.labs.arxiv.org/html/2305.07243/assets/figures/tortoise_diagram.png)

## 📊 关键结果 / 评测

- 开源 TTS 社区广泛采用
- 质量在当时（2023 初）显著优于其他开源方案

## 💡 借鉴意义（一句话）

做 TTS 的人了解——Tortoise 证明了 scaling + multi-stage generation 在开源 TTS 中的有效性，虽然推理速度慢但质量高。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2305.07243
- PDF: [[assets/papers/Tortoise.pdf|本地 PDF]]
- 源目录: `TTS-LLM/totorise .pdf`
