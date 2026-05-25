---
type: concept
aliases: [Rotary Position Embedding, 旋转位置编码]
---

# RoPE

## 定义
Rotary Position Embedding（旋转位置编码），通过对 query/key 向量施加与位置相关的旋转矩阵来编码相对位置信息。

## 数学形式

$$
\text{RoPE}(x_m, m) = x_m e^{im\theta}
$$

其中 $m$ 为位置，$\theta$ 为频率参数。

## 核心要点
1. 兼顾绝对位置和相对位置信息
2. 已成为主流 LLM 的标配位置编码（LLaMA、Qwen、Mistral 等）
3. 在语音模型中也被广泛使用（如 CosyVoice 2 的 Encoder_1）

## 代表工作
- [[Qwen2.5]]: LLM backbone 使用 RoPE
- [[CosyVoice2]]: 语音 tokenizer 的 Encoder_1 使用 RoPE

## 相关概念
- [[Transformer]]
