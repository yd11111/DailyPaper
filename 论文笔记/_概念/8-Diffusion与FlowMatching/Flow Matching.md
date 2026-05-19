---
type: concept
aliases: [CFM, Conditional Flow Matching]
---

# Flow Matching

## 定义

一种 generative modeling 范式：训练一个向量场 $v_\theta(x, t)$ 让其匹配从噪声分布到数据分布的概率路径，推理时用 ODE 积分还原数据。Conditional Flow Matching (CFM) 把它做条件化。比 [[DDPM]] 更直接、采样更少步即可。

## 数学形式

CFM 训练目标：
$$
\mathcal{L}_{\mathrm{CFM}} = \mathbb{E}_{t, x_0 \sim p_0, x_1 \sim p_1} \big\| v_\theta(x_t, t) - (x_1 - x_0) \big\|^2,\quad x_t = (1-t) x_0 + t x_1
$$

## 核心要点

1. 比扩散 noise prediction 更"线性"，容易训练
2. 1-4 步采样就能得到不错效果（rectified flow 极端情况 1 步）
3. [[Voicebox]] / [[F5-TTS]] / [[CosyVoice]] / [[CoVoMix2]] 都基于此

## 代表工作

- Lipman et al. (2022)
- [[F5-TTS]] / [[CosyVoice]] / [[Voicebox]]
- [[VibeVoice]]: 未用 Flow Matching；选择了 [[DDPM]] + [[DPM-Solver]] 路线

## 相关概念

- [[DDPM]]
- [[CosyVoice]]
- [[F5-TTS]]
