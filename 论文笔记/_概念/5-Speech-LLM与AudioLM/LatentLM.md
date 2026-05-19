---
type: concept
aliases: [Multimodal Latent Language Modeling]
---

# LatentLM

## 定义

微软 2024 年提出的多模态 latent 语言建模框架（Sun, Bao, Wang et al.）：用统一的 [[Next-Token Diffusion]] 范式建模**任意连续模态**——AR LLM 输出 hidden state，每个 token 接一个轻量 diffusion head 去噪出 latent。在视觉、语音、统一多模态任务上展示了通用能力。

## 数学形式

$$
p_\theta(z_{1:T}) = \prod_{t=1}^{T} p_\theta(z_t \mid h_t),\qquad h_t = \mathrm{LLM}(z_{<t})
$$
其中 $p_\theta(z_t | h_t)$ 由 diffusion head 实现（不是 softmax）。

## 核心要点

1. **AR 输出 hidden、扩散去 latent** 的统一形态
2. 用 σ-VAE 防止 AR 时 latent 方差坍缩
3. [[VibeVoice]] 是 LatentLM 在长篇 TTS 上的直接应用

## 代表工作

- 原论文 (arXiv 2412.08635)
- [[VibeVoice]]: TTS 直接搬用 LatentLM 范式

## 相关概念

- [[Next-Token Diffusion]]
- [[Diffusion Head]]
- [[VAE]]
- [[MAR]]
