---
title: "RepCodec: A Speech Representation Codec for Speech Tokenization"
method_name: "RepCodec"
authors: [Zhichao Huang, Chutong Meng, Tom Ko]
year: 2023
venue: arXiv
arxiv_id: "2309.00169"
pdf_path: "assets/papers/RepCodec.pdf"
library_source: "高德文献库"
source_topic: "speech-codec"
tags: [classic, codec]
created: 2026-05-22
---

# RepCodec: Speech Representation Codec

## 📌 一句话

提出在 **SSL representation 空间**（而非波形空间）做 VQ 量化——先用 HuBERT/WavLM 提取连续 representation，再训练 VQ-VAE 把它压缩为离散 token，比传统 audio codec tokenization（EnCodec/SoundStream）更适合语义驱动的 Speech LLM。

## 🛠 核心方法

**输入 → 输出**: SSL continuous representation → discrete semantic tokens → reconstructed representation

**架构组件**（按数据流顺序）:
1. **Frozen SSL Encoder**: HuBERT / WavLM 提取连续 representation
2. **RepCodec Encoder**: 轻量 CNN 下采样
3. **VQ Quantizer**: 单层 VQ 量化到离散码本
4. **RepCodec Decoder**: CNN 上采样重建 SSL representation

**关键创新**: 把 tokenization 目标从"重建波形"改为"重建 SSL representation"——这让 token 保留更多语义信息，下游 ASR/LM 任务效果更好（虽然不能直接重建波形）。

## 🖼 架构图

![Figure 1: RepCodec model architecture](https://ar5iv.labs.arxiv.org/html/2309.00169/assets/x1.png)

## 📊 关键结果 / 评测

- Decoder-only ASR: WER 2.87%（vs k-means 4.55% / EnCodec 5.31%）
- 语音重合成 ASR: WER 4.71%（vs k-means 7.61%）
- 码本利用率: 99.8%（vs k-means ~60-80%），几乎无 codebook collapse

## 💡 借鉴意义（一句话）

做 Speech Tokenizer / Speech LLM 的人关注——RepCodec 指出了"语义 tokenizer"和"声学 tokenizer"的分工：语义任务用 RepCodec，生成任务用 audio codec。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2309.00169
- PDF: [[assets/papers/RepCodec.pdf|本地 PDF]]
- 源目录: `speech-codec/RenCodec.pdf`
