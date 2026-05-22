---
title: "Mega-TTS 2: Boosting Prompting Mechanisms for Zero-Shot Speech Synthesis"
method_name: "MegaTTS2"
authors: [Ziyue Jiang, Jinglin Liu, Yi Ren, Jinzheng He, Zhenhui Ye]
year: 2024
venue: ICLR 2024
arxiv_id: "2307.07218"
pdf_path: "assets/papers/MegaTTS2.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, tts]
created: 2026-05-22
---

# Mega-TTS 2: Boosting Prompting for Zero-Shot TTS

## 📌 一句话

[[MegaTTS]] 的升级版——引入 **Multi-Reference Timbre Encoder (MRTE)** 从多段参考语音提取更鲁棒的 timbre 表示，PLM（Prosody Language Model）增强韵律建模，在 zero-shot TTS 上显著提升 speaker similarity。

## 🛠 核心方法

**输入 → 输出**: phoneme + multiple speech prompts → waveform

**架构组件**（按数据流顺序）:
1. **MRTE (Multi-Reference Timbre Encoder)**: 从多段参考语音提取 timbre，attention-based 聚合
2. **Global Timbre Encoder (GE)**: 全局说话人特征
3. **PLM (Prosody Language Model)**: 自回归韵律生成，增强版 P-LLM
4. **Acoustic Decoder**: content + timbre + prosody → mel → waveform

**关键创新**: 通过 **multi-reference prompting**（多段参考音频）和改进的 timbre encoder，解决了单段 prompt 不稳定的问题，显著提升 zero-shot speaker similarity。

## 🖼 架构图

![Figure 1: Mega-TTS 2 总览——MRTE + PLM 架构](https://ar5iv.labs.arxiv.org/html/2307.07218/assets/x1.png)

## 📊 关键结果 / 评测

- LibriSpeech: speaker similarity SECS 0.75+
- 优于 VALL-E / Mega-TTS 在 zero-shot 场景

## 💡 借鉴意义（一句话）

做 zero-shot TTS 的人参考——multi-reference prompting 是提升 speaker similarity 的有效策略。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2307.07218
- PDF: [[assets/papers/MegaTTS2.pdf|本地 PDF]]
- 源目录: `TTS-LLM/MegaTTS2.pdf`
