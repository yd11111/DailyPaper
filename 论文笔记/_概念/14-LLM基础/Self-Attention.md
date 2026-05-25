---
type: concept
aliases: [Self-Attention, 自注意力, Multi-Head Self-Attention, MHSA]
---

# Self-Attention

## 定义
Transformer 的核心机制，序列中每个位置都与所有其他位置计算注意力权重，从而捕捉全局依赖关系。Multi-Head 版本将隐状态拆分为多个 head 并行计算，再拼接投影。

## 数学形式

$$
\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right) V
$$

- $Q, K, V$: 查询、键、值矩阵，由输入线性投影得到
- $d_k$: 键向量维度（缩放因子）

## 核心要点
1. 计算复杂度 $O(n^2 d)$，对长序列开销大
2. 可捕捉任意距离的依赖关系（相比 CNN 的局部感受野）
3. FastSpeech 的 FFT block 中 self-attention 与 1D Conv 互补：前者建模全局，后者建模局部
4. 不同于 cross-attention（encoder-decoder attention），self-attention 的 Q/K/V 来自同一序列

## 代表工作
- [[Transformer]]: Vaswani et al. 2017 -- Attention Is All You Need
- [[FastSpeech]]: FFT block 中使用 2-head self-attention

## 相关概念
- [[Transformer]]
- [[Attention Alignment]]
- [[Layer Normalization]]
