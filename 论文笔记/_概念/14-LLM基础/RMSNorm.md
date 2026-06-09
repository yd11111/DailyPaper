---
type: concept
aliases: [Root Mean Square Normalization, 均方根归一化]
---

# RMSNorm

## 定义
Root Mean Square Layer Normalization，一种简化版 [[LayerNorm]]。省去均值中心化步骤，只做 RMS 归一化：$\hat{x} = x / \text{RMS}(x) \cdot \gamma$。计算量更小，效果接近。

## 数学形式

$$
\text{RMSNorm}(x) = \frac{x}{\sqrt{\frac{1}{d}\sum_{i=1}^{d} x_i^2 + \epsilon}} \cdot \gamma
$$

## 核心要点
1. 比 LayerNorm 少一次均值计算，训练/推理更快
2. 现代 LLM（LLaMA, Qwen, Gemma 等）的标准选择
3. 常与 QK-Norm 配合使用稳定注意力

## 代表工作
- LLaMA 系列: 首批大规模采用
- [[Qwen2.5]]: LLM backbone 使用
- [[dots-tts]]: AR-FM Head DiT 中使用 RMSNorm + QK-norm

## 相关概念
- [[RoPE]]
- [[Qwen2.5]]
