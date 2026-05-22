---
title: "Scaling Speech Tokenizers with Diffusion Autoencoders"
method_name: "ScalingSpeechTokenizers"
authors: []
year: 2026
venue: ICLR 2026 submission
arxiv_id: null
pdf_path: "assets/papers/ScalingSpeechTokenizers.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, codec]
created: 2026-05-22
---

# Scaling Speech Tokenizers with Diffusion Autoencoders

## 📌 一句话

提出用 **diffusion autoencoder** 替代传统 VQ/RVQ 做 speech tokenizer——在离散化阶段用 diffusion 模型做编解码，解决了 VQ 的码本利用率低和信息瓶颈问题，token 数量可 scaling。

## 🛠 核心方法

**输入 → 输出**: speech → diffusion-encoded tokens → reconstructed speech

**架构组件**:
1. **Diffusion Encoder**: 将语音编码为 latent（不做 VQ hard quantize）
2. **Discrete Bottleneck**: 在 diffusion latent 上做离散化
3. **Diffusion Decoder**: 从离散 token 重建语音

**关键创新**: 用 **diffusion autoencoder** 替代 VQ-VAE / RVQ 做 tokenizer——diffusion 的连续去噪过程比 VQ 的硬量化更温和，信息保留更多。

## 📊 关键结果 / 评测

- ICLR 2026 submission（anonymous）
- 同等 token 数下重建质量优于 VQ/RVQ-based tokenizer

## 💡 借鉴意义（一句话）

做 Speech Tokenizer 的人关注——diffusion autoencoder 是 VQ/RVQ 之外的第三条 tokenization 路线。

## 🔗 链接

- PDF: [[assets/papers/ScalingSpeechTokenizers.pdf|本地 PDF]]
- 源目录: `TTS-LLM/SCALING SPEECH TOKENIZERS WITH DIFFUSION AU- TOENCODERS.pdf`
