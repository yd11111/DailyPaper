---
title: "CLEAR: Continuous Latent Autoregressive Modeling for High-quality and Low-latency Speech Synthesis"
method_name: "CLEAR"
authors: [Chun Yat Wu, Jiajun Deng, Guinan Li, Qiuqiang Kong, Simon Lui]
year: 2025
venue: arXiv
arxiv_id: "2508.19098"
pdf_path: "assets/papers/CLEAR.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, tts]
created: 2026-05-22
---

# CLEAR: Continuous Latent AR for Low-Latency TTS

## 📌 一句话

港中大/华为提出的**连续 latent 自回归 TTS**——不做 VQ 离散化，直接在连续 latent 空间自回归建模（类似 [[MELLE]] 但方法不同），追求高质量 + 低延迟。

## 🛠 核心方法

**输入 → 输出**: text + reference → continuous latent → waveform

**架构组件**:
1. **Continuous Latent Encoder**: 将语音映射到连续 latent 空间
2. **AR Transformer**: 在连续 latent 上自回归生成
3. **Waveform Decoder**: latent → waveform

**关键创新**: 在连续 latent 空间做 AR 的同时，优化了推理延迟——通过高效的 latent 设计实现低延迟流式合成。

## 📊 关键结果 / 评测

- 高质量 zero-shot TTS
- 低延迟推理

## 💡 借鉴意义（一句话）

做 TTS 的人了解——CLEAR 是"连续 latent AR"方向的又一个数据点，与 [[MELLE]] / [[LatentLM]] 形成趋势。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2508.19098
- PDF: [[assets/papers/CLEAR.pdf|本地 PDF]]
- 源目录: `TTS-LLM/CLEAR- Continuous Latent Autoregressive Modeling for High-quality and Low-latency Speech Synthesis.pdf`
