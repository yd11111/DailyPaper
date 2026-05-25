---
type: concept
aliases: [因果卷积, Causal Conv]
---

# Causal Convolution

## 定义
一种确保模型在时间步 $t$ 的预测只依赖于 $t$ 及之前的输入的卷积操作，通过平移卷积输出或使用掩码实现。适用于自回归音频/序列生成。

## 数学形式

$$
y_t = \sum_{k=0}^{K-1} w_k \cdot x_{t-k}
$$

- 输入: 1-D 序列 $\mathbf{x}$
- 输出: 与输入等长的序列 $\mathbf{y}$，每个位置仅看到过去
- 感受野: 线性增长，$= (\text{layers}) \times (K - 1) + 1$

## 核心要点
1. 保证自回归性质：预测不依赖未来信息
2. 感受野随层数线性增长，对长序列建模效率低
3. 训练时所有时间步可并行计算（不同于 RNN）
4. 可通过空洞（dilation）扩展为 [[Dilated Causal Convolution]] 实现指数级感受野增长

## 代表工作
- [[WaveNet]]: 首次在原始波形生成中使用因果卷积

## 相关概念
- [[Dilated Causal Convolution]]
- [[Dilated Convolution]]
