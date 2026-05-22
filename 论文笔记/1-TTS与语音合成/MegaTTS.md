---
title: "Mega-TTS: Zero-Shot Text-to-Speech at Scale with Intrinsic Inductive Bias"
method_name: "MegaTTS"
authors: [Ziyue Jiang, Yi Ren, Zhenhui Ye, Jinglin Liu, Chen Zhang]
year: 2023
venue: arXiv
arxiv_id: "2306.03509"
pdf_path: "assets/papers/MegaTTS.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, tts]
created: 2026-05-22
---

# Mega-TTS: Zero-Shot TTS with Intrinsic Inductive Bias

## 📌 一句话

浙大/字节提出的 zero-shot TTS——将语音解耦为 content / timbre / prosody 三部分，用 **Prosody LLM (P-LLM)** 自回归建模韵律，VQGAN 处理声学细节，利用语音本身的归纳偏置减少对大规模数据的依赖。

## 🛠 核心方法

**输入 → 输出**: phoneme + speech prompt → waveform

**架构组件**（按数据流顺序）:
1. **Content Encoder**: phoneme → content representation
2. **Timbre Encoder**: 从 prompt 提取全局 timbre embedding
3. **P-LLM (Prosody Language Model)**: 自回归预测离散韵律 code
4. **VQGAN Decoder**: content + timbre + prosody → mel → waveform

**关键创新**: 显式利用了语音的**归纳偏置**（content/timbre/prosody 自然解耦），让每个模块只需建模一个维度，比端到端 codec LM（如 VALL-E）更数据高效。

## 🖼 架构图

![Figure 1: Mega-TTS 总览——VQGAN + Prosody LLM 架构](https://ar5iv.labs.arxiv.org/html/2306.03509/assets/x1.png)

## 📊 关键结果 / 评测

- LibriSpeech: speaker similarity 和自然度优于 VALL-E
- 数据效率: 20K 小时即可达到较好效果

## 💡 借鉴意义（一句话）

做 TTS 的人参考——Mega-TTS 的属性解耦思路（与 [[NaturalSpeech3]] 的 FACodec 类似）是 zero-shot TTS 的重要设计范式。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2306.03509
- PDF: [[assets/papers/MegaTTS.pdf|本地 PDF]]
- 源目录: `TTS-LLM/megaTTS.pdf`
