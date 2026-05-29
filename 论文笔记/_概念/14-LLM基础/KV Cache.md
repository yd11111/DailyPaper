---
type: concept
aliases: [KV Cache, KV-Cache, KV cache, K/V cache, Key-Value Cache]
domain: LLM
tags: [transformer, inference-optimization, self-attention, ar-decoder]
created: 2026-05-29
last_updated: 2026-05-29
---

# KV Cache

## 定义

自回归 Transformer 推理时的**关键优化**：把每一步生成产生的 self-attention key/value 张量缓存下来，下一步推理只对**新 token** 计算 K/V 并追加到 cache，避免对历史 token 重复计算。复杂度从每步 $O(t^2)$ 降到 $O(t)$。

## 数学形式

第 $l$ 层、第 $h$ 个 head 的 cache：

$$
\bm{K}^{(l,h)} \in \mathbb{R}^{T \times d_k}, \quad \bm{V}^{(l,h)} \in \mathbb{R}^{T \times d_v}
$$

每步新 token $t$：
1. 计算新 $\bm{k}_t, \bm{v}_t$
2. 追加：$\bm{K}^{(l,h)} \leftarrow [\bm{K}^{(l,h)}; \bm{k}_t]$，$\bm{V}$ 同理
3. attention 计算只对最新 query $\bm{q}_t$ 与整个 cache 做内积

## 核心要点

1. **必须按层 + 按 head 维护**，是 Transformer 推理的事实标配。
2. **内存随序列长度线性增长**——长上下文场景（128K+ tokens）会成为内存瓶颈，催生 [[Sliding-Window Attention]] / paged attention / KV cache quantization 等优化。
3. 可以被**主动改写**：除常规追加外，cache 还可以做 swap / drop / quantize / 跨 decoder 复用——这些 inference-time intervention 是新兴控制手段。

## 在音频领域的应用

- [[StyleSelfReferencing]] (2026) 提出 [[KV-Cache Swap]]——在 AR TTS decoder 中跨两个 decoder 复制 K/V cache，用来实现训练-free 的句内风格切换。
- 流式 TTS / 流式 LLM 中 [[Sliding-Window Attention]] + [[Attention Sink]] 配合 KV cache 截断使用。

## 代表工作（相关优化方向）

- FlashAttention (Dao et al.)：底层重新排列内存访问，与 KV cache 协同
- PagedAttention (vLLM): 把 KV cache 分页管理减少碎片
- [[Attention Sink]] (Xiao et al. 2024): 保留初始 KV + 滑窗，实现长序列推理
- KV cache quantization: INT8/INT4 量化 KV cache

## 相关概念

- [[Self-Attention]]
- [[Cross-Attention]]
- [[Sliding-Window Attention]]
- [[Attention Sink]]
- [[KV-Cache Swap]]
- [[Autoregressive]]
