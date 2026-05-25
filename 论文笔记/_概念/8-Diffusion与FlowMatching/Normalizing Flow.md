---
type: concept
aliases: [Normalizing Flow, NF, 正规化流]
---

# Normalizing Flow

## 定义
一类通过可逆变换将简单分布（如高斯）映射到复杂分布的生成模型。变换必须可逆且 Jacobian 行列式可高效计算，从而精确计算似然。

## 数学形式

$$
\log p(x) = \log p(z_0) - \sum_{i=1}^{K} \log \left|\det \frac{\partial f_i}{\partial z_{i-1}}\right|
$$

其中 $x = f_K \circ f_{K-1} \circ \cdots \circ f_1(z_0)$，$z_0 \sim \mathcal{N}(0, I)$

## 核心要点
1. 可逆变换链：每一步都是双射，推理时反向变换即可采样
2. Residual Coupling Layer 是 VITS/Glow-TTS 中常用的构造块
3. 相比 Diffusion/Flow Matching，NF 要求严格可逆，表达力受限但推理快
4. 在 VITS 中用于桥接先验（文本编码）和后验（音频编码）

## 代表工作
- [[VITS]]: 用 Residual Coupling Block 做先验-后验变换
- [[GPT-SoVITS]]: 继承 VITS 的 Flow 架构

## 相关概念
- [[Flow Matching]]
- [[Conditional Flow Matching]]
- [[VAE]]
- [[KL Divergence]]
