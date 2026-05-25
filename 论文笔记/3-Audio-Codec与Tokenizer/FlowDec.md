---
title: "FlowDec: A Flow-Based Full-Band General Audio Codec with High Perceptual Quality"
method_name: "FlowDec"
authors: [Simon Welker, Matthew Le, Ricky T.Q. Chen, Wei-Ning Hsu, Timo Gerkmann, Alexander Richard]
year: 2025
venue: ICLR 2025
arxiv_id: null
pdf_path: "assets/papers/FlowDec.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, codec]
created: 2026-05-22
---

# FlowDec: Flow-Based Full-Band Audio Codec

## 📌 一句话

Meta 提出的 **flow-based 音频 codec**——用 flow matching 替代传统的 HiFi-GAN 判别器训练 decoder，在全频带（48kHz）音频上实现更高的感知质量，同时保持低比特率。

## 🛠 核心方法

**输入 → 输出**: audio waveform → discrete tokens → reconstructed waveform (48kHz)

**架构组件**:
1. **Encoder**: 全卷积下采样
2. **RVQ**: 残差向量量化
3. **Flow-based Decoder**: 用 flow matching 做 decoder（替代 GAN）
4. **Full-band**: 支持 48kHz 全频带音频

**关键创新**: 用 **flow matching decoder** 替代传统 GAN decoder——flow model 的对数似然训练比 GAN 更稳定，且在全频带音频上感知质量更好。

## 📊 关键结果 / 评测

- 匿名投稿（ICLR 2025 发表），具体数字见论文 Table 1-2

## 💡 借鉴意义（一句话）

做 Audio Codec 的人关注——FlowDec 证明了 flow-based decoder 是 GAN decoder 的有力替代方案。

## 🔗 链接

- PDF: [[assets/papers/FlowDec.pdf|本地 PDF]]
- 源目录: `TTS-LLM/FlowDec.pdf`
