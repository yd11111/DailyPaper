---
type: concept
aliases: [VQ, 向量量化]
---

# VQ

## 定义
Vector Quantization（向量量化），将连续表示映射到有限离散码本中最近的码字。是 audio codec 和语义 tokenizer 的核心组件。

## 数学形式

$$
\mu = \text{VQ}(h, C) = \arg\min_{c_n \in C} \|h - c_n\|_2
$$

## 核心要点
1. 码本大小通常 1024-8192，CosyVoice S3 tokenizer 用 4096
2. 训练方法：EMA 更新 / commitment loss / Gumbel-Softmax
3. RVQ 是多层级联 VQ，逐层量化残差

## 代表工作
- [[CosyVoice]]: 在 ASR encoder 中间插入 VQ 得到 supervised semantic token
- [[EnCodec]]: RVQ 多层量化

## 相关概念
- [[Vector Quantization]]
- [[RVQ]]
- [[EMA]]
- [[Discrete Audio Token]]
