---
title: "Conditional Variational Autoencoder with Adversarial Learning for End-to-End Text-to-Speech"
method_name: "VITS"
authors: [Jaehyeon Kim, Jungil Kong, Juhee Son]
year: 2021
venue: ICML
arxiv_id: "2106.06103"
pdf_path: "assets/papers/VITS.pdf"
library_source: "高德文献库"
source_topic: "TTS-core"
tags: [classic, tts]
created: 2026-05-22
---

# VITS: End-to-End TTS with VAE + GAN

## 📌 一句话

首个将 VAE、normalizing flow、对抗训练统一到单阶段端到端 TTS 框架的工作，不需要独立声码器，直接文本到波形。MOS 达到接近真人水平，是后续 VITS2 / MB-iSTFT-VITS / CosyVoice 等众多系统的基石。

## 🛠 核心方法

**输入 → 输出**: phoneme sequence → 16 kHz waveform

**架构组件**（按数据流顺序）:
1. **Text Encoder (Transformer)**: phoneme → hidden representations
2. **Posterior Encoder + Normalizing Flow**: 训练时从 linear spectrogram 提取 latent z；flow 将后验分布对齐到先验分布，推理时从先验采样
3. **Stochastic Duration Predictor**: 用 flow 建模音素时长分布（非确定性），推理时采样时长实现多样化韵律
4. **[[HiFi-GAN]] Decoder**: latent z → waveform，端到端联合训练

**关键创新**: 把 [[VAE]] 的重建目标与 [[GAN]] 的对抗目标结合——VAE 保证全局结构，GAN 保证波形细节；normalizing flow 在 latent 空间弥合 text-speech gap，避免了 one-to-many 问题。

## 🖼 架构图

![Figure 1: Training (a) and inference (b) procedure — 整体系统框图含 Posterior Encoder / Prior Encoder / Decoder / Discriminator](https://ar5iv.labs.arxiv.org/html/2106.06103/assets/x1.png)

## 📊 关键结果 / 评测

- LJ Speech: MOS 4.43（GT 4.44），几乎持平真人
- 单阶段推理，无需 mel + vocoder 两步
- 多说话人 VCTK 上 MOS 3.90（GT 4.29），multi-speaker 支持

## 💡 借鉴意义（一句话）

做 TTS 的人**必读**——VITS 是 2021 年后 TTS 系统的设计范式原点：VAE + flow + GAN 三合一框架几乎所有后续工作都在此基础上迭代。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2106.06103
- PDF: [[assets/papers/VITS.pdf|本地 PDF]]
- 源目录: `TTS-core/VITS.pdf`
