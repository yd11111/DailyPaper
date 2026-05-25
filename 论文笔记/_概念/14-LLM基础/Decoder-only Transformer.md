---
type: concept
aliases: [Decoder-Only, Causal Transformer, 纯解码器 Transformer]
---

# Decoder-only Transformer

## 定义
仅由 decoder 层组成的 Transformer 架构，所有 token 使用因果注意力掩码（只能看到左侧 token）。通过将输入和输出拼接为单一序列进行自回归建模。

## 核心要点
1. GPT 系列的核心架构，也是当前 LLM 主流
2. 输入和输出共享同一序列，用特殊分隔符区分
3. 相比 encoder-decoder，结构更简单但输入端缺少双向注意力
4. 在语音领域常用于 acoustic token 生成（输入 semantic + acoustic prompt，输出 acoustic target）

## 代表工作
- [[GPT]]: decoder-only 的开创性工作
- [[SPEAR-TTS]]: S2 使用 12 层 decoder-only Transformer 做 semantic -> acoustic token
- [[VALL-E]]: decoder-only 做 text -> acoustic token

## 相关概念
- [[Transformer]]
- [[Encoder-Decoder Transformer]]
- [[Autoregressive]]
