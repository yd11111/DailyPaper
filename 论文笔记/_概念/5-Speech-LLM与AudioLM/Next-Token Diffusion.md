---
type: concept
aliases: [Token-Level Diffusion, AR + Diffusion Head]
---

# Next-Token Diffusion

## 定义

把 LLM 的"下一 token"预测从 softmax over 离散 token 换成"扩散去噪到一个连续 latent"的范式：AR 主干输出 hidden $h_t$，token 级扩散头基于 $h_t$ 条件去噪生成下一连续 token $z_t$。由 [[LatentLM]]、[[MAR]] 等推动；**[[VibeVoice]] 把它直接用到长篇 TTS**。

## 数学形式

训练目标（DDPM 形式）：
$$
\mathcal{L} = \mathbb{E}_{t,\bm{\epsilon}} \big\| \bm{\epsilon} - \epsilon_\theta(z_t^{(\tau)}, \tau, h_t) \big\|^2
$$
推理：从 $\mathcal{N}(0, I)$ 起步，[[DPM-Solver|DPM-Solver++]] 几步采到 $z_t$。

## 核心要点

1. 让 AR LM 直接生成**连续**输出，绕开 RVQ / 多码本工程负担
2. 对长序列高度友好（连续 latent 可以做得很短）
3. CFG 引导可显著提质量
4. 已成现代统一多模态生成的主流范式之一

## 代表工作

- [[LatentLM]]
- [[MAR]] (Autoregressive Image Generation without Vector Quantization)
- [[VibeVoice]]: 长篇多说话人 TTS

## 相关概念

- [[LatentLM]]
- [[Diffusion Head]]
- [[Classifier-Free Guidance]]
- [[DDPM]]
