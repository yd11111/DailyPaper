---
title: "Emilia: An Extensive, Multilingual, and Diverse Speech Dataset for Large-Scale Speech Generation"
method_name: "Emilia"
authors: [Haorui He, Zengqiang Shang, Chaoren Wang, Xuyuan Li, Yicheng Gu]
year: 2024
venue: SLT 2024
arxiv_id: "2407.05361"
pdf_path: "assets/papers/Emilia.pdf"
library_source: "高德文献库"
source_topic: "Dataset"
tags: [classic, dataset]
created: 2026-05-22
---

# Emilia: 大规模多语言语音生成数据集

## 📌 一句话

港中深 + 阿里联合发布的 **101K 小时多语言**语音数据集（6 语种），配套开源数据处理流水线 Emilia-Pipe，专为大规模 TTS / Speech LLM 训练设计，是目前公开最大的语音生成训练数据集之一。

## 🛠 核心方法

**输入 → 输出**: raw audio → cleaned, segmented, annotated speech corpus

**架构组件**（按 pipeline 顺序）:
1. **Emilia-Pipe**: 自动化数据处理流水线——VAD → speaker diarization → ASR → quality filtering → dedup
2. **多语言覆盖**: English / Chinese / German / French / Japanese / Korean，共 101K 小时
3. **质量控制**: SNR / DNSMOS / ASR CER 多维度筛选
4. **开源**: 数据 + pipeline 代码全部开放

**关键创新**: 不只发布数据集，同时开源了**可复现的 pipeline**（Emilia-Pipe），让其他团队能用相同流程处理自有数据，降低了大规模 TTS 数据准备的门槛。

## 🖼 架构图

![Figure 1: Emilia-Pipe 处理流水线概览](https://ar5iv.labs.arxiv.org/html/2407.05361/assets/x1.png)

## 📊 关键结果 / 评测

- 总量 101K 小时（English 46K + Chinese 49K + 其他 4 语种 6K）
- 下游验证: 用 Emilia 训练的 TTS 模型在 naturalness 和 speaker similarity 上持平/优于 LibriTTS 训练的模型

## 💡 借鉴意义（一句话）

做 TTS / Speech LLM 数据准备的人**必读**——Emilia-Pipe 是目前最完整的开源语音数据处理 pipeline，可直接复用。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2407.05361
- PDF: [[assets/papers/Emilia.pdf|本地 PDF]]
- 源目录: `Dataset/Emilia-dataset.pdf`
