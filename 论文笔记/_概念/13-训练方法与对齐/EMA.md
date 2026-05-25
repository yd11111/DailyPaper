---
type: concept
aliases: [Exponential Moving Average, 指数移动平均]
---

# EMA

## 定义
Exponential Moving Average，指数移动平均。在 VQ 训练中用于更新码本向量，在模型训练中用于维护参数的平滑版本。

## 数学形式

$$
c := \alpha \cdot c + (1-\alpha) \cdot h
$$

## 核心要点
1. VQ 码本更新的标准方法，避免码本坍缩
2. 衰减系数 $\alpha$ 通常为 0.99 或 0.999
3. 也用于模型权重的 EMA 平均（如 diffusion model 的 EMA checkpoint）

## 代表工作
- [[CosyVoice]]: S3 Tokenizer 码本 EMA 更新

## 相关概念
- [[Vector Quantization]]
