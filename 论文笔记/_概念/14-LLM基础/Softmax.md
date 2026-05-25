---
type: concept
aliases: [Softmax 函数, Softmax Distribution]
---

# Softmax

## 定义
将 $K$ 维实数向量映射为概率分布的函数，广泛用于分类任务的输出层。

## 数学形式

$$
\text{softmax}(z_i) = \frac{e^{z_i}}{\sum_{j=1}^{K} e^{z_j}}
$$

## 核心要点
1. WaveNet 使用 256 维 softmax 输出音频采样值的分类分布
2. 论文指出 softmax 分布比混合模型更灵活，因为不假设分布形状
3. 是自回归语言模型的标准输出层

## 代表工作
- [[WaveNet]]: 256 级 softmax 建模 μ-law 量化后的音频采样

## 相关概念
- [[Mu-Law Companding]]
- [[Autoregressive Model]]
