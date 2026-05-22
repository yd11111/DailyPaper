---
title: "BASE TTS: Lessons from building a billion-parameter Text-to-Speech model on 100K hours of data"
method_name: "BaseTTS"
authors: [Mateusz Łajszczak, Arent van Korlaar, Guillermo Cámbara, Yang Li, Fatih Beyhan]
year: 2024
venue: arXiv
arxiv_id: "2402.08093"
pdf_path: "assets/papers/BaseTTS.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, tts]
created: 2026-05-22
---

# BASE TTS: Billion-Parameter TTS on 100K Hours

## 📌 一句话

Amazon 的**大规模 TTS**实验——1B 参数 AR model 在 100K 小时数据上训练，用 [[WavLM]]-based speech tokenizer 做离散化，核心贡献是 scaling 实验和 emergent abilities 的观察（大模型出现复合语句/情感表达等能力涌现）。

## 🛠 核心方法

**输入 → 输出**: text + reference speech → speech tokens → waveform

**架构组件**（按数据流顺序）:
1. **WavLM-based Speech Tokenizer**: 解耦 speaker/content 的离散化
2. **AR Model (GPT-like)**: text + reference → speech token 序列
3. **Speechcode Decoder**: speech token → waveform

**关键创新**: 系统性地研究了 TTS 的 **scaling law**——发现 1B 参数 + 100K 小时后出现"能力涌现"（compound sentences / emotions / foreign words 处理能力突然提升）。

## 🖼 架构图

![Figure 1: BASE TTS 总览——tokenizer + AR model + decoder](https://ar5iv.labs.arxiv.org/html/2402.08093/assets/figures/gpt.png)

## 📊 关键结果 / 评测

- 1B 参数模型在复合语句上显著优于小模型
- 出现 emergent abilities（类似 LLM 的能力涌现）
- 100K 小时训练数据

## 💡 借鉴意义（一句话）

做大规模 TTS 的人关注——BASE TTS 是首个系统性证明 TTS 存在 scaling-based emergent abilities 的工作。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2402.08093
- PDF: [[assets/papers/BaseTTS.pdf|本地 PDF]]
- 源目录: `TTS-LLM/BaseTTS.pdf`
