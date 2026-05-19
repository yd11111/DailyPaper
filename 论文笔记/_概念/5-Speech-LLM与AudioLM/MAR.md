---
type: concept
aliases: [Masked Autoregressive, Autoregressive Image Generation without Vector Quantization]
---

# MAR

## 定义

何恺明组 2024 年的《Autoregressive Image Generation without Vector Quantization》提出的方法：图像生成中放弃 VQ，转而 AR 预测连续 latent；每个位置用一个轻量 diffusion head 输出连续向量。是 [[Next-Token Diffusion]] / [[LatentLM]] 的同期开创性工作。

## 核心要点

1. 提出 **token-level diffusion head**：4 层小 MLP/Transformer 即可
2. 抛弃 [[RVQ]] / VQ 离散化，连续 latent 直接 AR 预测
3. [[VibeVoice]] 的 4 层 diffusion head 直接借鉴 MAR

## 代表工作

- 原论文 (arXiv 2406.11838, Li, Tian, Li, Deng, He 2024)
- [[VibeVoice]]: diffusion head 设计沿用 MAR

## 相关概念

- [[Next-Token Diffusion]]
- [[LatentLM]]
- [[Diffusion Head]]
