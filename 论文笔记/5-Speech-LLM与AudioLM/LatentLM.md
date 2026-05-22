---
title: "Multimodal Latent Language Modeling with Next-Token Diffusion"
method_name: "LatentLM"
authors: [Yutao Sun, Hangbo Bao, Wenhui Wang, Zhiliang Peng, Li Dong]
year: 2024
venue: arXiv
arxiv_id: "2412.08635"
pdf_path: "assets/papers/LatentLM.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, speech-llm]
created: 2026-05-22
---

# LatentLM: Multimodal Latent Language Modeling

## 📌 一句话

微软提出的**统一多模态语言模型**——用 **next-token diffusion** 在连续 latent 空间做自回归（不需要 VQ 离散化），σ-VAE 做 tokenizer，统一处理文本/图像/音频/视频，是"连续 token + diffusion 预测"范式的代表。

## 🛠 核心方法

**输入 → 输出**: text/image/audio/video (混合) → latent tokens → generation

**架构组件**（按数据流顺序）:
1. **σ-VAE Tokenizer**: 固定方差 VAE，将连续数据映射到 latent vectors（不做 VQ）
2. **Causal Transformer**: 统一的自回归 backbone
3. **Next-Token Diffusion**: 每步用小型 diffusion head 预测下一个连续 latent vector
4. **Modality-specific Decoders**: latent → 各模态输出

**关键创新**: 提出 **next-token diffusion**——不再预测离散 token，而是每步预测一个连续 latent vector（通过小 diffusion 采样），彻底绕开了 VQ 的信息瓶颈。σ-VAE 的固定方差设计避免了 variance collapse。

## 🖼 架构图

![Figure 2: LatentLM — σ-VAE tokenizer + causal transformer + next-token diffusion](https://ar5iv.labs.arxiv.org/html/2412.08635/assets/x2.png)

## 📊 关键结果 / 评测

- ImageNet FID: 2.24（479M 参数，优于 DiT-XL/2 的 2.27 / 675M）
- Text-to-Image FID: 14.54（Transfusion 16.10）
- LibriSpeech TTS: SIM 0.697, WER-H 1.8（优于 VALL-E 2 的 SIM 0.643）
- σ-VAE 语音压缩: 1600× 压缩率，PESQ 2.724

## 💡 借鉴意义（一句话）

做多模态 LM / Speech LLM 的人关注——LatentLM 展示了"连续 latent + diffusion prediction"替代 VQ 离散化的可行性。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2412.08635
- PDF: [[assets/papers/LatentLM.pdf|本地 PDF]]
- 源目录: `TTS-LLM/LantentLM.pdf`
