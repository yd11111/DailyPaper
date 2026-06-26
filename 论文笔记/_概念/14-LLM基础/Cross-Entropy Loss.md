---
type: concept
aliases: [交叉熵损失, CE Loss, Negative Log-Likelihood]
---

# Cross-Entropy Loss

## 定义

分类/序列建模中最基本的训练损失函数，衡量模型预测分布与真实标签之间的差异。在自回归语言模型中，对每个位置计算预测 token 概率的负对数似然。

## 数学形式

$$
\mathcal{L} = -\sum_{t=1}^{T} \log p_\theta(y_t \mid y_{<t}, x)
$$

- 输入：模型在每个时间步的预测概率分布 $p_\theta$
- 输出：标量损失值
- $y_t$: 第 $t$ 个目标 token
- $T$: 序列长度

## 核心要点
1. 自回归 LM 的标准训练目标，也广泛用于 codec-LM TTS（如 [[VALL-E]]、[[Sarashina22-TTS]]）
2. 在 TTS 中通常仅在语音 token 位置计算（不对文本 token 部分 backprop）
3. 多任务训练时常与其他 loss（重建 loss / KL loss）加权组合

## 代表工作
- [[VALL-E]]: 在离散 codec token 上使用 CE loss
- [[Sarashina22-TTS]]: 在语义 token 位置计算 CE loss

## 相关概念
- [[Language Modeling]]
- [[Autoregressive Model]]
