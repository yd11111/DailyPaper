---
type: concept
aliases: [Layer Normalization, LayerNorm, LN, 层归一化]
---

# Layer Normalization

## 定义
一种归一化技术，对单个样本在特征维度上进行归一化（与 Batch Normalization 在 batch 维度归一化不同），是 Transformer 架构中的标准组件。

## 数学形式

$$
\text{LayerNorm}(x) = \gamma \cdot \frac{x - \mu}{\sqrt{\sigma^2 + \epsilon}} + \beta
$$

- $\mu, \sigma^2$: 在特征维度上计算的均值和方差
- $\gamma, \beta$: 可学习的缩放和偏移参数

## 核心要点
1. 不依赖 batch 统计量，适合变长序列和小 batch 训练
2. Transformer 中每个子层后接 LayerNorm（Post-LN）或子层前接 LayerNorm（Pre-LN）
3. FastSpeech 的 FFT block 和 Duration Predictor 中均使用 LayerNorm

## 代表工作
- Ba et al. 2016: Layer Normalization 原始论文
- [[Transformer]]: 标准使用 LayerNorm
- [[FastSpeech]]: FFT block 中使用

## 相关概念
- [[Transformer]]
- [[Feed-Forward Transformer]]
