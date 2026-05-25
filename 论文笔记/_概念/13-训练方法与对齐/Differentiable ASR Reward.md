---
type: concept
aliases: [可微分ASR奖励, Differentiable ASR Loss]
---

# Differentiable ASR Reward

## 定义
一种通过 Gumbel softmax 使离散 token 采样可微分，然后用冻结的 ASR 后端计算文本重建概率作为奖励信号的训练方法。比 DPO 更高效且泛化能力更强。

## 数学形式

$$
L_{ASR} = -\log P(Y|\hat{H};\theta_{ASR})
$$

其中 $\hat{H} = \text{Proj}_{up}(\text{GumbelSoftmax}(\text{logits}))$。

## 核心要点
1. 不需要构建偏好对，直接优化文本可懂度
2. 通过 Gumbel softmax 绕过离散采样的不可微问题
3. 在 out-of-domain 场景（如 SEED test set）比 DPO 泛化更好
4. 可与 DPO 互补使用，组合效果最佳

## 代表工作
- [[CosyVoice2]]: 首次在 TTS LM 中提出并验证此方法

## 相关概念
- [[DPO]]
- [[Straight-Through Estimator]]
