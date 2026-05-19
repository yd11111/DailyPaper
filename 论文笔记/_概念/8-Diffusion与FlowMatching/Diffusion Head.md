---
type: concept
aliases: [Token-level Diffusion Head]
---

# Diffusion Head

## 定义

挂在 AR 主干每个 token 位置上的小型扩散网络：以 LLM hidden $h_t$ 为条件，去噪生成连续 latent $z_t$；通常 3-6 层、参数量 << LLM。是 [[Next-Token Diffusion]] / [[LatentLM]] / [[MAR]] / [[VibeVoice]] 的核心组件。

## 数学形式

对位置 $t$：
$$
z_t = \text{Sample}\big(\epsilon_\theta(z_t^{(\tau)}, \tau, h_t)\big)
$$
通过 [[DPM-Solver|DPM-Solver++]] 等几步采样得到。

## 核心要点

1. **轻量级**：4 层即可（[[VibeVoice]] / [[MAR]]）
2. 训练用 [[DDPM]] 噪声预测 loss
3. 推理用 [[Classifier-Free Guidance|CFG]] + [[DPM-Solver]]
4. 把"AR 离散预测"改造成"AR 连续生成"的关键

## 代表工作

- [[MAR]]: 视觉
- [[LatentLM]]: 多模态
- [[VibeVoice]]: 长篇 TTS

## 相关概念

- [[Next-Token Diffusion]]
- [[LatentLM]]
- [[MAR]]
- [[DDPM]]
