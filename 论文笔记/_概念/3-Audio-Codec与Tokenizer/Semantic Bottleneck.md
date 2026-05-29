---
type: concept
aliases: [SemBo, 语义瓶颈]
---

# Semantic Bottleneck (SemBo)

## 定义

把高维语义编码器（典型如 1280-d MiDashengLM）的输出用一个轻量学习式 MLP 压缩到低维（典型 128-d），同时用"重建 + 时间相似度"双损失约束，得到适合作下游 supervision 的紧凑语义信号。由 [[LoSATok]] (Zhang et al., 2026) 提出。

## 数学形式

压缩器 $C$ 和恢复器 $R$ 均为 2-layer MLP w/ GELU：

$$
z_s^l = C(z_s^h), \quad \hat z_s^h = R(z_s^l), \quad z_s^l \in \mathbb{R}^{T \times 128}
$$

训练目标：

$$
\mathcal{L}_{\mathrm{SemBo}} = \lambda_{\mathrm{recon}} \mathcal{L}_{\mathrm{recon}} + \mathcal{L}_{\mathrm{tr}}
$$

其中 $\lambda_{\mathrm{recon}}=10^3$，$\mathcal{L}_{\mathrm{tr}}$ 为 [[Time-Relation Loss]]。

## 核心要点

1. **训练式压缩 vs training-free**：相比 PCA / Channel Merging，SemBo 学到非线性映射，在 FSC 等结构化语义任务上显著更优（FSC 89.01 vs CM 86.11 / PCA 78.06）。
2. **独立预训练**：在主 tokenizer 训练之前先单独训好 SemBo，整个 SemBo（含原 MiDashengLM 部分）都冻结。
3. **不是 VQ bottleneck**：SemBo 输出的是连续低维向量，不做离散化。
4. **目标只是 supervision 信号**：SemBo 自身不直接对音频解码，只为下游 tokenizer（如 LoSATok）提供低维 semantic target。

## 代表工作

- [[LoSATok]]: 首次提出，作为 dual-level semantic supervision 中的 low-dim source。

## 相关概念

- [[Time-Relation Loss]]：SemBo 的关键 supervision 项。
- [[Dual-Level Semantic Supervision]]：LoSATok 主训练用 SemBo 输出做监督的方法。
- [[MiDashengLM]]：SemBo 默认压缩的语义源。
- [[Audio VAE]]：通用 audio latent 类比。
