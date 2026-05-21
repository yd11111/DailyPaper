---
title: "HuBERT: Self-Supervised Speech Representation Learning by Masked Prediction of Hidden Units"
method_name: "HuBERT"
authors: [Wei-Ning Hsu, Benjamin Bolte, Yao-Hung Hubert Tsai, Kushal Lakhotia, Ruslan Salakhutdinov, Abdelrahman Mohamed]
year: 2021
venue: arXiv
arxiv_id: "2106.07447"
pdf_path: "assets/papers/HuBERT.pdf"
library_source: "高德文献库"
source_topic: "SSL"
tags: [classic, ssl]
created: 2026-05-19
---

# HuBERT: Hidden-Unit BERT

## 📌 一句话

语音自监督学习里程碑工作。用**离线 k-means 聚类**生成对齐目标，配 BERT-style masked prediction loss 训语音表示。是后续语义 token 路线（[[AudioLM]] / [[SpeechGPT]] / [[VALL-E]] 一类）的事实标准 SSL backbone。

## 🛠 核心方法

**输入 → 输出**: 原始语音波形 → 连续帧级表示（也可下游 k-means 离散化成语义 token）

**架构组件**（按数据流顺序）:
1. **CNN Encoder**: 7 层 512-channel 卷积，把波形下采样到 20ms / frame
2. **Mask 替换**: 随机选 frame span 替换成 `[MSK]` token（mask span l = 10）
3. **Transformer**: BASE / LARGE / X-LARGE 三档；预测 mask 处的 hidden unit
4. **Acoustic Unit Discovery 系统**（**离线**，不在主网络里）: 用 k-means 在 MFCC 或上一轮表示上聚类，输出每帧的 pseudo-label `z₁, z₂, ...`
5. **训练目标**: 只在 mask 位置上算交叉熵（cross-entropy loss vs k-means pseudo-label）

**关键创新**: 依赖聚类的**一致性**而非**正确性**——即使第一轮 k-means 标签很噪（100 cluster 从 MFCC 聚），只要 mask 预测一致，模型也能学好表示；再用学到的表示重新聚类，迭代 2 轮提升标签质量。

## 🖼 架构图

![Fig.1 (p2): HuBERT 用 CNN encoder → Transformer 预测被 mask 帧的离线 k-means 聚类标签](assets/papers/figs/HuBERT_fig1.png)

## 📊 关键结果 / 评测

- **LibriSpeech 960h fine-tune**: 匹配或超过当时 SOTA [[wav2vec 2.0]]
- **Libri-light 60kh 预训练 + 1B 模型**: dev-other / test-other 上相对 [[wav2vec 2.0]] **WER 降 19% / 13%**
- **少量标注 robust**: 在 10min / 1h / 10h / 100h / 960h 子集都有效

## 💡 借鉴意义（一句话）

做 Speech LLM / 离散语音 token 的人**必读**——HuBERT k-means 出的语义 token 是 [[AudioLM]] / [[SpeechGPT]] 一类的事实输入，离散 token TTS（VALL-E 之后所有变种）的语义分支也常基于 HuBERT。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2106.07447
- PDF: [[assets/papers/HuBERT.pdf|本地 PDF]]
- 源目录: `SSL/hubert.pdf`
