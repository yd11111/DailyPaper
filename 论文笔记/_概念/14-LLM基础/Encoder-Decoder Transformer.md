---
type: concept
aliases: [Encoder-Decoder, Seq2Seq Transformer]
---

# Encoder-Decoder Transformer

## 定义
由 encoder（双向自注意力）和 decoder（因果自注意力 + cross-attention）组成的 Transformer 架构。Encoder 编码完整输入序列，decoder 自回归生成输出序列并通过 cross-attention 关注 encoder 输出。

## 核心要点
1. Encoder 双向看完整输入；decoder 因果生成 + 通过 cross-attention 读取 encoder
2. 适合输入输出模态/长度不同的 seq2seq 任务（翻译、TTS text->token、ASR）
3. 典型代表：T5、BART、原始 Transformer (Vaswani 2017)
4. 与 decoder-only 相比，encoder 的双向注意力对输入的建模更充分

## 代表工作
- [[T5]]: 经典 encoder-decoder 预训练模型
- [[SPEAR-TTS]]: S1 使用 T5-Large encoder-decoder 做 text -> semantic token
- Vaswani et al. (2017): 原始 Transformer 论文

## 相关概念
- [[Transformer]]
- [[Cross-Attention]]
- [[Decoder-only Transformer]]
