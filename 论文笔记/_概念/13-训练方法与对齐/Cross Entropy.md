---
type: concept
aliases: [CE Loss, 交叉熵损失]
---

# Cross Entropy

## 定义
分类任务的标准损失函数，衡量预测概率分布与真实标签之间的差异。在 LLM-based TTS 中用于训练自回归语义 token 预测。

## 数学形式

$$
\mathcal{L}_{CE} = -\frac{1}{L} \sum_{l=1}^{L} \log q(\mu_l)
$$

## 核心要点
1. LLM-based TTS 的标准训练损失
2. CosyVoice 仅在 speech token 和 EOS token 上计算 CE，不在文本 token 上计算

## 代表工作
- [[CosyVoice]]: LLM 训练损失
- [[VALL-E]]: AR 阶段训练损失

## 相关概念
- [[Autoregressive Model]]
