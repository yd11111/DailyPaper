---
title: "NaturalSpeech: End-to-End Text to Speech Synthesis with Human-Level Quality"
method_name: "NaturalSpeech"
authors: [Xu Tan, Jiawei Chen, Haohe Liu, Jian Cong, Chen Zhang]
year: 2022
venue: TPAMI
arxiv_id: "2205.04421"
pdf_path: "assets/papers/NaturalSpeech.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, tts]
created: 2026-05-22
---

# NaturalSpeech: End-to-End TTS with Human-Level Quality

## 📌 一句话

微软提出的端到端 TTS 系统——基于 VAE + flow + memory 机制，在 LJSpeech 上首次达到**人类水平**的 MOS 分数（-0.01 vs 录音），核心贡献是用 bidirectional prior/posterior + memory-augmented VAE 缩小 prior-posterior gap。

## 🛠 核心方法

**输入 → 输出**: phoneme sequence → waveform

**架构组件**（按数据流顺序）:
1. **Phoneme Encoder**: 文本到 phoneme hidden sequence
2. **Differentiable Durator**: 可微分时长预测 + 上采样
3. **Bidirectional Prior/Posterior**: flow-based 模块缩小 text prior 与 speech posterior 的分布差距
4. **VAE with Memory**: 码本增强的 VAE decoder，减少 one-to-many 映射难度

**关键创新**: 系统性地分析并解决了 TTS 中 prior-posterior gap 的四个来源（phoneme encoding / duration / acoustic variation / decoder），每个 gap 配对一个专门模块。

## 🖼 架构图

![Figure 1: NaturalSpeech — 系统总览（phoneme encoder → durator → bidirectional prior/posterior → waveform decoder）](https://ar5iv.labs.arxiv.org/html/2205.04421/assets/x1.png)

## 📊 关键结果 / 评测

- LJSpeech: MOS 4.58（录音 4.59），首次无统计显著差异
- CMOS: +0.01 vs GT recording

## 💡 借鉴意义（一句话）

做 TTS 的人了解——NaturalSpeech 是首个宣称达到人类水平的 TTS 系统，后续 [[NaturalSpeech2]] / [[NaturalSpeech3]] 在此基础上扩展到 zero-shot 和多说话人。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2205.04421
- PDF: [[assets/papers/NaturalSpeech.pdf|本地 PDF]]
- 源目录: `TTS-LLM/NS.pdf`
