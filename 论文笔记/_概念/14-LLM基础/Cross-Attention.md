---
type: concept
aliases: [Cross-Attention, 交叉注意力]
---

# Cross-Attention

## 定义
Transformer 中的一种注意力机制，Query 来自一个模态/序列，Key 和 Value 来自另一个模态/序列，用于跨模态或跨序列的信息融合。

## 数学形式

$$
\text{CrossAttn}(Q, K, V) = \text{softmax}\left(\frac{Q K^\top}{\sqrt{d_k}}\right) V
$$

其中 $Q = W_Q \cdot \mathbf{h}_{\text{source}}$，$K = W_K \cdot \mathbf{h}_{\text{target}}$，$V = W_V \cdot \mathbf{h}_{\text{target}}$

## 核心要点
1. Q 来自"需要融合信息"的一方，K/V 来自"提供信息"的一方
2. 在 TTS 中常用于将文本信息注入音频特征（或反向）
3. 区别于 Self-Attention（Q/K/V 来自同一序列）

## 代表工作
- [[GPT-SoVITS]]: MRTE 用 Cross-Attention 融合 SSL 内容编码和文本编码
- [[Transformer]]: 经典 encoder-decoder 架构的核心组件

## 相关概念
- [[Transformer]]
- [[Autoregressive Model]]
