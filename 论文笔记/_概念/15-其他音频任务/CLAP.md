---
type: concept
aliases: [Contrastive Language-Audio Pretraining, LAION-CLAP, Microsoft CLAP]
---

# CLAP

## 定义

Contrastive Language-Audio Pretraining，CLIP 的音频版本：对 (音频, 文本) pair 做 InfoNCE 对比学习，让音频与其描述在共享嵌入空间靠近。微软原版 (Elizalde et al., 2022) 与后续 LAION-CLAP 是两条主线。广泛用作 [[Text-to-Audio]] 模型的文本条件编码器、TTA / audio captioning 的自动评测器（CLAP score）、以及 LALM 的音频 encoder。

## 数学形式

batch 内对 $N$ 对 (audio, text)：
$$\mathcal{L}_{\text{CLAP}} = -\frac{1}{2N}\sum_{i=1}^N \bigl[\log \tfrac{e^{s(a_i, t_i)/\tau}}{\sum_j e^{s(a_i, t_j)/\tau}} + \log \tfrac{e^{s(a_i, t_i)/\tau}}{\sum_j e^{s(a_j, t_i)/\tau}}\bigr]$$
其中 $s(\cdot,\cdot)$ 为 cosine similarity。

## 核心要点

1. **跨模态对齐**：把音频与文本投到同一空间
2. **三大用途**：(a) TTA 条件编码 (b) 评测 score (c) LALM audio encoder
3. **LAION-CLAP**：开源，训练数据 633k+ pair，效果常超原版
4. **CLAP score**：常作为 TTA 生成质量自动评测之一，但有"自评偏差"问题

## 代表工作

- 原论文 (arXiv 2206.04769)
- LAION-CLAP (arXiv 2211.06687)
- [[AudioLDM]] 用 CLAP 做文本条件
- [[WavFlow]] 用 CLAP 做客观评测

## 评测/常见数字

- ESC-50 zero-shot 分类：LAION-CLAP top-1 ≈ 88%
- AudioCaps audio-to-text retrieval R@1 ≈ 35%

## 相关概念

- [[CLIP]]
- [[Text-to-Audio]]
- [[AudioLDM]]
- [[Stable Audio]]
- [[LALM]]
