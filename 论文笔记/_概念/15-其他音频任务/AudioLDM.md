---
type: concept
aliases: [Audio LDM, AudioLDM2]
---

# AudioLDM

## 定义

Surrey 大学 / 微软 2023 年提出的 text-to-audio latent diffusion 模型 (Liu et al., ICML 2023)。把 [[Stable Diffusion]] 思路搬到音频域：用 audio VAE 压到 latent → 在 latent 上跑文本条件扩散 → VAE decoder 还原波形。是 [[Text-to-Audio]] 任务的开山级开源工作。

## 数学形式

文本 $y$ 经 [[CLAP]] / T5 编码 → $c$；latent $z_0 \sim q(z_0|x)$（来自 audio VAE）。在 latent 上跑 DDPM：
$$\nabla_\theta \mathbb{E}_{t,\epsilon}\bigl[\|\epsilon - \epsilon_\theta(z_t, t, c)\|^2\bigr]$$
推理时反向采样得 $z_0$，再用 VAE decoder 解到波形。

## 核心要点

1. **latent diffusion**：直接学波形/STFT 代价过大，搬到 latent 后训练 + 推理皆友好
2. **CLAP 条件**：用 [[CLAP]] 把文本和音频对齐的特征做扩散条件，效果好于 T5
3. **后续 AudioLDM 2**：加入 GPT-style audio LM 做语义层 priors
4. **限制**：长 form (> 10 秒) 一致性差，被 [[Stable Audio]] 反超

## 代表工作

- 原论文 ICML 2023 (arXiv 2301.12503)
- AudioLDM 2 (arXiv 2308.05734)
- [[Target-KL-VAE]]：作为下游验证 VAE 质量的 TTA 平台之一

## 评测/常见数字

- AudioCaps FAD：AudioLDM-Large ~1.7，IS ~8.1

## 相关概念

- [[Stable Audio]]
- [[Text-to-Audio]]
- [[Audio VAE]]
- [[CLAP]]
- [[Flow Matching]]
