---
type: concept
aliases: [Direct Preference Optimization, 直接偏好优化]
---

# DPO

## 定义
Direct Preference Optimization，一种无需训练奖励模型的偏好对齐方法，直接用偏好数据对（preferred vs. rejected）优化策略模型。

## 数学形式

$$
L_{DPO}(\pi_\theta;\pi_{ref}) = -\log\sigma\left(\beta\log\frac{\pi_\theta(y^w|x)}{\pi_{ref}(y^w|x)} - \beta\log\frac{\pi_\theta(y^l|x)}{\pi_{ref}(y^l|x)}\right)
$$

- $y^w, y^l$: 偏好 / 非偏好样本
- $\pi_{ref}$: SFT 参考策略
- $\beta$: 温度参数

## 核心要点
1. 比 RLHF + PPO 更简单稳定，无需额外奖励模型
2. 在 TTS 中可基于 speaker similarity 或 MOS 构建偏好对
3. 对 hard cases（如重复文本）的偏好对构建需要谨慎，否则可能引入噪声

## 代表工作
- [[CosyVoice2]]: 基于说话人相似度构建 DPO 偏好对优化 TTS LM

## 相关概念
- [[Straight-Through Estimator]]
- [[Differentiable ASR Reward]]
