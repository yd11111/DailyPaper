---
title: "WavLM: Large-Scale Self-Supervised Pre-Training for Full Stack Speech Processing"
method_name: "WavLM"
authors: [Sanyuan Chen, Chengyi Wang, Zhengyang Chen, Yu Wu, Shujie Liu]
year: 2021
venue: IEEE JSTSP
arxiv_id: "2110.13900"
pdf_path: "assets/papers/WavLM.pdf"
library_source: "高德文献库"
source_topic: "SSL"
tags: [classic, ssl]
created: 2026-05-22
---

# WavLM: Full Stack Speech SSL

## 📌 一句话

微软提出的语音 SSL 模型，在 [[HuBERT]] 基础上加入**去噪预训练**（输入带噪/混叠语音，预测干净语音的 pseudo label），使模型同时擅长内容理解和说话人/噪声相关任务，在 [[SUPERB]] 全 10 个任务上取得 SOTA。

## 🛠 核心方法

**输入 → 输出**: raw waveform → contextualized speech representations

**架构组件**（按数据流顺序）:
1. **CNN Feature Encoder**: 波形 → 帧级特征（同 HuBERT）
2. **Gated Relative Position Bias**: 改进的 Transformer 位置编码
3. **Denoising Pre-training**: 输入加噪/混叠 → 预测干净语音的 HuBERT pseudo label
4. **Masked Prediction**: 标准 mask-then-predict 目标（同 HuBERT）

**关键创新**: 去噪预训练目标让模型在学习内容表示的同时保留了说话人/噪声信息——HuBERT 擅长内容但丢弃非内容信息，WavLM 两者兼顾。

## 🖼 架构图

![Figure 1: WavLM model architecture — denoising + masked prediction pre-training](https://ar5iv.labs.arxiv.org/html/2110.13900/assets/x1.png)

## 📊 关键结果 / 评测

- SUPERB leaderboard 全 10 任务 SOTA（发布时）
- 特别在 SE / SS / SV 等非内容任务上大幅领先 HuBERT

## 💡 借鉴意义（一句话）

做语音 SSL / 说话人表示 / 语音增强的人**必读**——WavLM 是目前最均衡的语音 SSL 模型，在需要同时处理内容+说话人信息的场景下首选。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2110.13900
- PDF: [[assets/papers/WavLM.pdf|本地 PDF]]
- 源目录: `SSL/WavLM.pdf`
