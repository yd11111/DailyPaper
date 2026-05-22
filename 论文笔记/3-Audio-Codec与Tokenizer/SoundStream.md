---
title: "SoundStream: An End-to-End Neural Audio Codec"
method_name: "SoundStream"
authors: [Neil Zeghidour, Alejandro Luebs, Ahmed Omran, Jan Skoglund, Marco Tagliasacchi]
year: 2021
venue: IEEE/ACM TASLP
arxiv_id: "2107.03312"
pdf_path: "assets/papers/SoundStream.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, codec]
created: 2026-05-22
---

# SoundStream: End-to-End Neural Audio Codec

## 📌 一句话

Google 提出的**端到端神经音频编解码器**——encoder-RVQ-decoder 架构 + 对抗训练，在 3kbps 即可达到 Opus 12kbps 的质量，是 [[EnCodec]] / [[DAC]] 等后续 codec 和 [[AudioLM]] semantic-acoustic 分层的直接基础。

## 🛠 核心方法

**输入 → 输出**: audio waveform → RVQ discrete tokens → reconstructed waveform

**架构组件**（按数据流顺序）:
1. **Encoder**: 全卷积下采样（strided conv），将波形映射到 latent
2. **Residual Vector Quantizer (RVQ)**: 多层级量化，每层量化上一层残差
3. **Decoder**: 全卷积上采样重建波形
4. **Discriminator**: 多尺度 + multi-period 判别器做对抗训练

**关键创新**: 首次将 **RVQ（Residual VQ）** 引入神经音频 codec——相比单层 VQ，RVQ 可以用少量码本实现高质量量化，且支持可变比特率（丢弃高层 RVQ = 降低码率）。

## 🖼 架构图

![Figure 2: SoundStream — encoder + RVQ + decoder 架构](https://ar5iv.labs.arxiv.org/html/2107.03312/assets/x2.png)

## 📊 关键结果 / 评测

- 3 kbps: MUSHRA 与 Opus 12 kbps 相当
- 支持 speech/music/环境音的统一编解码
- 可变比特率: 1.5-18 kbps

## 💡 借鉴意义（一句话）

做 Audio Codec / Speech Tokenizer 的人**必读**——SoundStream 确立了 encoder-RVQ-decoder + GAN 训练的标准 codec 架构，后续 [[EnCodec]] / [[DAC]] / [[SNAC]] 均为其变体。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2107.03312
- PDF: [[assets/papers/SoundStream.pdf|本地 PDF]]
- 源目录: `TTS-LLM/SoundsStream.pdf`
