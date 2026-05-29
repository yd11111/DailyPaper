---
type: concept
aliases: [Attention Sink, attention sink, StreamingLLM sink]
domain: LLM
tags: [attention, transformer, long-context, streaming, ar-decoder]
created: 2026-05-29
last_updated: 2026-05-29
based_on:
  - "Xiao-2024-StreamingLLM"
---

# Attention Sink

## 定义

Xiao et al. (2024, "Efficient streaming language models with attention sinks") 发现的现象：AR Transformer 推理时，**早期几个 token（即使是无意义的 BOS / 空格）会被后续 token 大量注意到**，成为 attention 分布的"汇"。若在长序列 streaming 中丢弃这些 token，模型会快速崩溃；保留它们 + 滑窗截断中间段则可稳定运行。

## 核心机制

1. **softmax 归一化**强制 attention 总分布为 1；当后续 token 实际"无需查询过去"时，多余的 attention 权重会**自然汇入早期固定位置**。
2. 早期 token 因此承担了"attention dump site"角色，本身的语义贡献可能不重要，但**作为系统平衡点必须保留**。
3. 任何 AR Transformer 都会自发形成此现象，包括 LLaMA / Mistral / GPT 系。

## 实务方案：StreamingLLM

- 保留**初始 $k$ 个 token**（sink，通常 $k=4$）
- 用 [[Sliding-Window Attention]] 截断中间，只保留最近 $w$ 个 token
- 推理时可在 $O(w)$ 内存下 stream 任意长序列

## 在音频领域的关联

- [[StyleSelfReferencing]] (Kang et al. 2026) 命名的 [[Style Self-Referencing]] 现象与 attention sink **同源**——都是"AR decoder 早期 token 主导后续生成"的特化表现。区别：
  - Attention Sink 是**通用语言建模**层面的观察
  - Style Self-Referencing 是其在 **prompt-based TTS 风格控制**子场景的具体表现 + 反制方案
- 流式 TTS / 流式 ASR 的低延迟推理同样会用 attention sink + 滑窗

## 代表工作

- Xiao et al. (2024) "Efficient streaming language models with attention sinks" — 原始发现
- [[StyleSelfReferencing]] (2026) — 在 TTS 控制语境下的延伸

## 相关概念

- [[Self-Attention]]
- [[Sliding-Window Attention]]
- [[KV Cache]]
- [[KV-Cache Swap]]
- [[Style Self-Referencing]]
