---
title: "Towards Naturalistic Voice Conversion: NaturalVoices Dataset with an Automatic Speaker Evaluation Metric"
method_name: "NaturalVoices"
authors: []
year: 2024
venue: arXiv
arxiv_id: "2406.04494"
pdf_path: "assets/papers/NaturalVoices.pdf"
library_source: "高德文献库"
source_topic: "tools"
tags: [classic, dataset, vc]
created: 2026-05-22
---

# NaturalVoices: VC Evaluation Dataset + Pipeline

## 📌 一句话

面向 Voice Conversion 评测的**自然语音数据集**，配套自动化数据采集 pipeline（VAD / ASR / speaker recognition / emotion / SNR），以及自动 speaker 评估 metric。

## 🛠 核心方法

**输入 → 输出**: 数据集 + 评估工具

**架构组件**（按 pipeline 顺序）:
1. **Data Sourcing Pipeline**: diarization → ASR → speaker recognition → gender/age → emotion → SNR
2. **NaturalVoices Dataset**: 比 VCTK 更自然的语音数据（情感维度覆盖更广）
3. **Automatic Speaker Evaluation**: 自动化说话人相似度评估 metric

**关键创新**: 构建了比 VCTK 情感覆盖更广的 VC 评测数据集，且开源了端到端数据采集 pipeline。

## 🖼 架构图

![Figure 1: Data sourcing pipeline — 多模块自动化数据采集流程](https://ar5iv.labs.arxiv.org/html/2406.04494/assets/x1.png)

## 📊 关键结果 / 评测

- NaturalVoices 在 arousal/dominance/valence 维度上覆盖范围显著大于 VCTK

## 💡 借鉴意义（一句话）

做 VC 评测的人参考——NaturalVoices 提供了更真实的评测场景，自动 pipeline 可复用。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2406.04494
- PDF: [[assets/papers/NaturalVoices.pdf|本地 PDF]]
- 源目录: `tools/2406.04494v1.pdf`
