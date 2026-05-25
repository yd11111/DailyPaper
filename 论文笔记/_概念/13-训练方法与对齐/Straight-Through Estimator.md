---
type: concept
aliases: [STE, 直通估计器]
---

# Straight-Through Estimator (STE)

## 定义

一种用于不可微离散操作（如取整 round、argmax）的梯度近似方法：前向传播时执行离散操作，反向传播时直接将梯度"直通"传递，忽略离散化步骤的零梯度。广泛用于量化感知训练和离散表示学习。

## 数学形式

$$
\text{Forward: } y = \text{round}(x), \quad \text{Backward: } \frac{\partial \mathcal{L}}{\partial x} \approx \frac{\partial \mathcal{L}}{\partial y}
$$

- 前向：$y = \text{round}(x)$（离散化）
- 反向：$\nabla_x \mathcal{L} = \nabla_y \mathcal{L}$（直通，假设 $\partial y / \partial x = 1$）

## 核心要点

1. **解决离散操作不可微问题**: round / argmax 的真实梯度几乎处处为零，STE 用恒等映射近似替代
2. **有偏但实用**: 理论上是有偏估计，但实践中对 [[Vector Quantization|VQ]]、[[FSQ]]、二值化网络等训练效果良好
3. **在 VoxCPM 中的应用**: [[FSQ]] 层的 round 操作通过 STE 实现端到端梯度反传，使量化瓶颈参与整体优化

## 代表工作

- **Bengio et al. 2013**: STE 原始提出 (Estimating or Propagating Gradients Through Stochastic Neurons)
- [[VoxCPM]]: FSQ 瓶颈通过 STE 实现端到端训练
- [[CosyVoice2]]: FSQ 模块训练时使用 STE 进行梯度近似

## 相关概念

- [[FSQ]]: 使用 STE 的标量量化方案
- [[RVQ]]: 传统 VQ 也常用 STE 或其变体
- [[Vector Quantization]]: VQ-VAE 中的 STE 应用
