---
type: concept
aliases: [Rectified Linear Unit, 线性整流单元]
---

# ReLU

## 定义
最常用的深度学习激活函数，$f(x) = \max(0, x)$。简单高效，但在音频建模中被 [[Gated Activation Unit]] 显著超越。

## 数学形式

$$
f(x) = \max(0, x)
$$

## 核心要点
1. 计算简单、缓解梯度消失
2. 在 WaveNet 实验中效果显著不如门控激活
3. 变体包括 Leaky ReLU、PReLU、GELU、SiLU 等

## 相关概念
- [[Gated Activation Unit]]
- [[Softmax]]
