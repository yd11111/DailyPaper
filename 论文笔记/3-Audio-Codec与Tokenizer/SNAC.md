---
title: "SNAC: Multi-Scale Neural Audio Codec"
method_name: "SNAC"
authors: [Hubert Siuzdak, Florian Grötschla, Luca A. Lanzendörfer]
year: 2024
venue: arXiv
arxiv_id: "2410.14411"
pdf_path: "assets/papers/SNAC.pdf"
library_source: "高德文献库"
source_topic: "speech-codec"
tags: [classic, codec]
created: 2026-05-22
---

# SNAC: Multi-Scale Neural Audio Codec

## 📌 一句话

提出 **Multi-Scale RVQ**——不同 RVQ 层在不同时间分辨率上做量化（粗层低帧率 / 细层高帧率），比标准 RVQ 在相同比特率下重建质量更好，同时对 AR 语言模型更友好（token 序列更短）。

## 🛠 核心方法

**输入 → 输出**: audio waveform → multi-scale discrete tokens → reconstructed waveform

**架构组件**（按数据流顺序）:
1. **Encoder**: CNN 下采样到 latent
2. **Multi-Scale RVQ**: 第 1 层在最低帧率量化（粗粒度），后续层逐层提升帧率（细粒度）
3. **Decoder**: 从 multi-scale tokens 重建波形
4. **Adversarial Training**: 多尺度判别器

**关键创新**: 打破了传统 RVQ "所有层同帧率"的假设——粗层捕捉整体结构（低帧率足够），细层补充高频细节（需要高帧率），更高效地利用比特预算。

## 🖼 架构图

![Figure 1: 标准 RVQ (a) vs Multi-Scale RVQ (b) 对比](https://ar5iv.labs.arxiv.org/html/2410.14411/assets/x1.png)

## 📊 关键结果 / 评测

- MUSHRA: 在 speech 和 music 上优于同比特率的 [[DAC]] / [[EnCodec]]
- 对 AR LM 友好: 第一层 token 序列长度减半

## 💡 借鉴意义（一句话）

做 Audio Codec / Speech LLM 的人**必读**——SNAC 的 multi-scale 思路解决了 RVQ token 序列过长的痛点，已被多个 Speech LLM 项目采用。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2410.14411
- PDF: [[assets/papers/SNAC.pdf|本地 PDF]]
- 源目录: `speech-codec/Snac.pdf`
