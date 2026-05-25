---
type: concept
aliases: [CFM, Conditional FM, 条件流匹配]
---

# Conditional Flow Matching

## 定义
在 Flow Matching 框架下引入条件信息（如说话人、文本、语义 token）的生成方法。通过学习从高斯噪声到目标分布的连续向量场，实现高效采样。

## 数学形式

$$
\mathcal{L}_{CFM} = \mathbb{E}_{t, p_0(X_0), q(X_1)} \left\| \omega_t(\phi_t(X_0, X_1)|X_1) - \nu_t(\phi_t(X_0, X_1)|\theta) \right\|
$$

Optimal Transport 路径：$\phi_t^{OT} = (1-(1-\sigma)t)X_0 + tX_1$

## 核心要点
1. 比 DDPM 训练更简单（直线路径 vs 复杂 SDE）
2. 推理更快（更少步数即可收敛）
3. OT-CFM 进一步优化路径为 optimal transport
4. 常配合 Classifier-Free Guidance 使用

## 代表工作
- [[CosyVoice]]: OT-CFM 生成 Mel 频谱
- [[Voicebox]]: mask-and-infill 的 Flow Matching TTS
- [[F5-TTS]]: 纯 Flow Matching TTS

## 相关概念
- [[Flow Matching]]
- [[OT-CFM]]
- [[DDPM]]
- [[Classifier-Free Guidance]]
