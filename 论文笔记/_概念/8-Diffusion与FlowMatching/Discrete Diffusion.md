---
type: concept
aliases: [Masked Diffusion, 离散扩散, D3PM, MDLM]
---

# Discrete Diffusion

## 定义

在离散 token 空间中进行扩散建模的生成方法。不同于连续扩散在实数空间加高斯噪声，离散扩散通过 masking（替换为 [MASK] token）或 corruption（随机替换为其他 token）模拟前向过程，反向过程逐步恢复原始 token。

## 数学形式

**前向过程（masking-based）:**

$$
\mathbf{X}_t = \mathbf{X} \odot \mathbf{M}_t, \quad m_{t,i} \sim \text{Bernoulli}(\sigma(t))
$$

**训练损失:**

$$
\mathcal{L} = \mathbb{E}_{t} \left[ -\sum_{i} m_{t,i} \cdot \log p_\theta(x_i \mid \mathbf{X}_t) \right]
$$

**反向过程:** 从全 [MASK] 开始，每步预测所有被 mask 的 token，保留高置信度预测，将低置信度的重新 mask。

## 核心要点
1. 前向过程等价于 Masked Language Model，但有显式的噪声调度 $\sigma(t)$
2. 反向过程迭代式，通常 4-20 步即可生成高质量结果（远少于连续扩散的 50-1000 步）
3. 天然适合离散 codec token 空间的语音/音频生成
4. 推理时可用 confidence-based re-masking 策略（类似 MaskGIT）

## 代表工作
- [[NaturalSpeech3]]: 在四个属性子空间中分别做离散扩散
- [[MaskGCT]]: 用 masked generative codec Transformer 做 TTS
- [[SoundStorm]]: 离散 token 的并行解码

## 相关概念
- [[DDPM]]
- [[Flow Matching]]
- [[Classifier-Free Guidance]]
- [[RVQ]]
