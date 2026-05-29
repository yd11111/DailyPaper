---
title: "Autoregressive Speech Synthesis without Vector Quantization"
method_name: "MELLE"
authors: [Lingwei Meng, Long Zhou, Shujie Liu, Sanyuan Chen, Bing Han]
year: 2024
venue: arXiv
arxiv_id: "2407.08551"
pdf_path: "assets/papers/MELLE.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, tts]
created: 2026-05-22
---

# MELLE: AR Speech Synthesis without VQ

## 📌 一句话

微软提出的**无 VQ 自回归 TTS**——直接在连续 mel-spectrogram 空间做自回归生成（不做离散化），用 **latent sampling module** 替代 VQ 的离散瓶颈，避免了 VQ 的码本坍塌和信息损失问题。

## 🛠 核心方法

**输入 → 输出**: text + acoustic prompt → continuous mel-spectrogram → waveform

**架构组件**（按数据流顺序）:
1. **Decoder-only Transformer**: 自回归生成连续 mel frame
2. **Latent Sampling Module**: 变分采样（不是 VQ），生成连续 latent 再映射到 mel
3. **Stop Prediction Layer**: 预测序列结束
4. **Post-Net**: mel 后处理 + vocoder

**关键创新**: 证明了 **AR TTS 不必经过 VQ 离散化**——直接在连续空间自回归 + variational sampling 同样可以做高质量语音生成，且避免了码本设计的工程复杂度。

## 🖼 架构图

![Figure 1: MELLE — 连续 mel AR 生成 + Latent Sampling Module](https://ar5iv.labs.arxiv.org/html/2407.08551/assets/x1.png)

## 📊 关键结果 / 评测

- LibriSpeech: WER 和 speaker similarity 与 VALL-E 系列可比
- 无需 VQ 码本设计

## 💡 借鉴意义（一句话）

做 TTS / codec 设计的人关注——MELLE 挑战了"语音必须离散化才能做 AR"的假设，提供了连续空间 AR 的可行方案。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2407.08551
- PDF: [[assets/papers/MELLE.pdf|本地 PDF]]
- 源目录: `TTS-LLM/MELLE.pdf`

> 🔍 **对比报告**: [[2026-05-29-VALL-E系列演进调研]]
