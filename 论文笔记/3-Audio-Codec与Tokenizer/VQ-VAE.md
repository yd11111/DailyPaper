---
title: "Neural Discrete Representation Learning"
method_name: "VQ-VAE"
authors: [Aaron van den Oord, Oriol Vinyals, Koray Kavukcuoglu]
year: 2017
venue: NeurIPS 2017
arxiv_id: "1711.00937"
pdf_path: "assets/papers/VQ-VAE.pdf"
library_source: "高德文献库"
source_topic: "speech-codec"
tags: [classic, codec]
created: 2026-05-22
---

# VQ-VAE: Neural Discrete Representation Learning

## 📌 一句话

DeepMind 奠基性工作——提出 **Vector Quantised VAE**，在 VAE 的 latent 空间加入离散码本量化，用 straight-through estimator + commitment loss 训练。是后续所有 audio codec（[[EnCodec]] / [[SoundStream]] / [[DAC]]）和 discrete speech token 的理论基础。

## 🛠 核心方法

**输入 → 输出**: input (image/audio) → discrete latent codes → reconstruction

**架构组件**（按数据流顺序）:
1. **Encoder**: 将输入映射到连续 latent embedding
2. **Vector Quantization**: 最近邻查找码本中最接近的向量，替换为 codebook entry
3. **Decoder**: 从离散码解码重建
4. **Training**: reconstruction loss + VQ loss (commitment + codebook) + straight-through estimator

**关键创新**: 首次在生成模型中引入**可学习的离散瓶颈**——VAE 的连续 latent 改为离散码本索引，使生成模型可以与自回归 prior 结合，开创了"先量化再自回归生成"的范式。

## 🖼 架构图

![Figure 1: VQ-VAE — encoder → quantize → decoder + embedding space visualization](https://ar5iv.labs.arxiv.org/html/1711.00937/assets/figures/Figure1_9.png)

## 📊 关键结果 / 评测

- 图像: 在 CIFAR-10 / ImageNet 上重建质量接近连续 VAE
- 语音: 对 VCTK 语音做 VQ，学到的离散 code 自动对应 phoneme-like 单元
- WaveNet decoder: VQ-VAE + WaveNet 做 unconditional speech generation

## 💡 借鉴意义（一句话）

做 Audio Codec / Speech Tokenizer 的人**必须了解**——VQ-VAE 是整个"离散语音 token"技术栈的理论起点，[[RVQ]] / [[FSQ]] / codebook 设计都源于此。

## 🔗 链接

- arXiv: https://arxiv.org/abs/1711.00937
- PDF: [[assets/papers/VQ-VAE.pdf|本地 PDF]]
- 源目录: `speech-codec/VQ_VAE.pdf`
