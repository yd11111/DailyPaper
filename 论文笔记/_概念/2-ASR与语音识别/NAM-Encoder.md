---
type: concept
aliases: [Neural Associative Memory Encoder, NAM Encoder]
---

# NAM-Encoder

## 定义

ASR contextual biasing 中的一种 bias encoder 设计——把每条 bias phrase（关键词、专有名词等）写进一种「神经联想记忆」结构，让解码器可以基于声学查询 attend 到匹配的 key 上。与 [[CLAS]] 同属 attention-based biasing 派系，但联想记忆形式让 retrieval 更高效，可扩展到上千条 bias 列表。

## 核心要点

1. **本质**：每个 bias phrase → (key, value) 对，写入可微分记忆
2. **查询**：解码器隐状态作为 query，soft retrieve 出最相关的 bias context
3. **可扩展**：相对 [[CLAS]] 在 bias 列表很大时性能掉得更慢
4. **训练**：常带 `<bias>` / `</bias>` token 让模型显式标注是否在引用 bias
5. **典型对比**：与 [[Trie-based biasing]] / [[CLAS]] / [[CTC-WS-Streaming]] 并列为四大 contextual ASR 方法

## 代表工作

- Munkhdalai et al. 等 Google contextual ASR 系列

## 评测/常见数字

- 在 1000+ bias phrase 测试集上 WER 相对 no-bias 降低 20–40%

## 相关概念

- [[CLAS]]
- [[Trie-based biasing]]
- [[CTC-WS-Streaming]]
- [[Conformer]]
