---
title: "FELLE: Autoregressive Speech Synthesis with Token-Wise Coarse-to-Fine Flow Matching"
method_name: "FELLE"
authors: [Hui Wang, Shujie Liu, Lingwei Meng, Jinyu Li, Yifan Yang]
year: 2025
venue: arXiv
arxiv_id: "2502.11128"
pdf_path: "assets/papers/FELLE.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, tts]
created: 2026-05-22
---

# FELLE: AR TTS with Token-Wise Coarse-to-Fine Flow Matching

## 📌 一句话

微软/南开提出的 AR TTS——在 [[MELLE]] 的连续 mel AR 基础上引入 **token-wise coarse-to-fine flow matching**，每个 AR step 用小型 flow matching 生成更精细的 mel frame，兼顾 AR 的序列建模能力和 flow matching 的生成质量。

## 🛠 核心方法

**输入 → 输出**: text + acoustic prompt → continuous mel frames → waveform

**架构组件**:
1. **AR Transformer**: 自回归生成粗粒度 mel representation
2. **Token-wise Flow Matching**: 每个 AR step 内用 flow matching 精细化
3. **Vocoder**: mel → waveform

**关键创新**: 将 flow matching 嵌入到 AR 的**每一步**——AR 提供全局序列结构，flow matching 提供局部精细度，两者互补。

## 📊 关键结果 / 评测

- Continuation: WER-C 1.53%, SIM-o 0.513（MELLE SIM-o 0.480）
- Cross-sentence: WER-C 2.20%, SIM-o 0.619（MELLE 0.591）
- Cross-sentence MOS: 4.157（超过 GT 的 4.043）

## 💡 借鉴意义（一句话）

做 AR TTS 的人关注——FELLE 展示了 AR + flow matching 的混合架构是提升连续 mel AR 质量的有效方案。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2502.11128
- PDF: [[assets/papers/FELLE.pdf|本地 PDF]]
- 源目录: `TTS-LLM/FELLE.pdf`
