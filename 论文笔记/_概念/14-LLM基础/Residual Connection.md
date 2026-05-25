---
type: concept
aliases: [残差连接, Skip Connection (residual), ResNet Connection]
---

# Residual Connection

## 定义
He et al. (2015) 提出的深度网络训练技巧，将层的输入直接加到输出上：$\mathbf{y} = F(\mathbf{x}) + \mathbf{x}$。使梯度可以直接回传，支持训练极深的网络。

## 数学形式

$$
\mathbf{y} = F(\mathbf{x}) + \mathbf{x}
$$

## 核心要点
1. 解决深层网络的梯度退化问题
2. WaveNet 中每个残差 block 都使用残差连接
3. Transformer 中也是核心组件（与 LayerNorm 配合）

## 代表工作
- [[WaveNet]]: 残差 block 加速收敛、支持深层堆叠

## 相关概念
- [[Skip Connection]]
