---
type: concept
aliases: [Rotary Position Embedding, 旋转位置编码, RoFormer]
---

# RoPE (Rotary Position Embedding)

## 定义

一种相对位置编码方法，通过旋转矩阵将绝对位置信息编码到注意力计算中，使注意力得分自然蕴含相对位置关系，支持长度外推。

## 数学形式

$$
f(q, m) = R_m q, \quad f(k, n) = R_n k
$$

$$
\langle f(q,m), f(k,n) \rangle = \langle R_{m-n} q, k \rangle
$$

其中 $R_m$ 是与位置 $m$ 相关的旋转矩阵。

## 核心要点

1. 相对位置信息通过旋转矩阵自然编码到 QK 内积中
2. 支持长度外推（训练短序列、推理长序列）
3. 被广泛采用于 LLaMA、Qwen、GPT-NeoX 等主流 LLM
4. CosyVoice 3 的 Voice Encoder 使用 RoPE

## 代表工作

- Su et al. 2024: "RoFormer: Enhanced transformer with rotary position embedding" (Neurocomputing)
- [[CosyVoice3]]: Voice Encoder₁ 的 12 层 Transformer 使用 RoPE

## 相关概念

- [[GPT]]
- [[Qwen2.5]]
