---
type: concept
aliases: [GRL, 梯度反转层, Domain-Adversarial Training]
---

# Gradient Reversal Layer

## 定义

梯度反转层：正向传播时作为恒等映射，反向传播时将梯度取反。用于对抗训练，迫使特征编码器学习对某个属性（如说话人身份）不变的表示。

## 数学形式

$$
\text{GRL}(\mathbf{x}) = \mathbf{x} \quad (\text{forward})
$$

$$
\frac{\partial \text{GRL}}{\partial \mathbf{x}} = -\lambda \mathbf{I} \quad (\text{backward})
$$

- $\lambda$: 梯度反转系数，控制对抗强度

## 核心要点

1. 最早由 Ganin et al. (2016) 提出，用于域自适应（domain adaptation）
2. 在语音领域常用于解耦说话人身份与其他属性（情感、内容等）
3. 无需额外训练判别器网络，直接通过梯度操作实现对抗

## 代表工作

- [[IndexTTS2]]: 用 GRL 实现情感特征与说话人音色的解耦

## 相关概念

- [[LoRA]]
