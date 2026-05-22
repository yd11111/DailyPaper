---
title: "Towards Controllable Speech Synthesis in the Era of Large Language Models: A Systematic Survey"
method_name: "ControllableTTS-Survey"
authors: [Tianxin Xie, Yan Rong, Pengfei Zhang, Wenwu Wang, Li Liu]
year: 2024
venue: arXiv
arxiv_id: "2412.06602"
pdf_path: "assets/papers/ControllableTTS-Survey.pdf"
library_source: "高德文献库"
source_topic: "ControllableTTS"
tags: [classic, tts, survey]
created: 2026-05-22
---

# ControllableTTS-Survey: LLM 时代的可控语音合成综述

## 📌 一句话

港科大团队系统梳理了 LLM 时代可控 TTS 的方法分类（从控制策略、网络结构、训练方式三个维度），覆盖 style/emotion/prosody/speaker/language 等多维控制，是当前最全面的可控 TTS 综述。

## 🛠 核心方法

**输入 → 输出**: 综述论文，无独立模型

**架构组件**（按分类维度）:
1. **Control Strategy Taxonomy**: prompt-based / instruction-based / attribute-conditioned / reference-based
2. **Network Structure**: AR codec LM / NAR diffusion / flow matching / hybrid
3. **Training Paradigm**: supervised / RL / in-context learning / zero-shot

**关键创新**: 首次从"控制策略"而非"模型架构"出发做分类，更好地揭示了 LLM-based TTS 与传统 TTS 在控制维度上的本质差异。

## 🖼 架构图

![Figure 2: General pipeline of controllable TTS — 从 network structure 角度的系统分类](https://ar5iv.labs.arxiv.org/html/2412.06602/assets/x2.png)

## 📊 关键结果 / 评测

- 综述，无实验结果
- 覆盖 2020-2024 年约 100+ 篇可控 TTS 论文

## 💡 借鉴意义（一句话）

做可控 TTS / 情感 TTS / prosody 控制的人的**入门必读 survey**——快速了解领域全貌和技术路线选择。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2412.06602
- PDF: [[assets/papers/ControllableTTS-Survey.pdf|本地 PDF]]
- 源目录: `ControllableTTS/Survey-ControllableTTS.pdf`
