---
type: concept
aliases: [标签平滑]
---

# Label Smoothing

## 定义
正则化技术，将 one-hot 标签 soft 化为 $(1-\epsilon)$ 的真实类别概率 + $\epsilon/(K-1)$ 的均匀噪声，防止模型对训练标签过度自信，提升泛化性能。

## 数学形式

$$
y'_k = (1-\epsilon) \cdot y_k + \frac{\epsilon}{K}
$$

- $\epsilon$: 平滑系数（常用 0.1）
- $K$: 类别数
- $y_k$: 原始 one-hot 标签

## 核心要点
1. 防止模型 logits 过大导致过拟合
2. 在 seq2seq 和分类任务中广泛使用
3. 典型值 $\epsilon = 0.1$

## 代表工作
- [[SPEAR-TTS]]: S1 微调时使用 label smoothing = 0.1
- Szegedy et al. (2016): 在 Inception-v2 中首次提出

## 相关概念
- [[Cross Entropy]]
- [[Transformer]]
