---
type: concept
aliases: [KV-Cache Swap, KV Cache Swap, KV cache swap, K/V swap]
domain: LLM
tags: [inference-time-intervention, kv-cache, ar-decoder, controllable-tts, training-free]
created: 2026-05-29
last_updated: 2026-05-29
based_on:
  - "Kang-2026-arXiv:2605.27376"
related_maps:
  - "[[TTS-技术路线图]]"
---

# KV-Cache Swap

## 定义

推理时介入手段：在 AR decoder 生成过程中的某个**过渡点** $t^*$，把当前 decoder 的 self-attention KV cache 的前 $n$ 个位置整段替换为由另一个 decoder（带不同条件 / prompt）单独跑出来的 KV。目标是**强行植入另一个上下文的"早期 anchor"**，触发 decoder 后续生成行为的切换。

## 数学形式

$$
\bm{K}^{(A)}_{1:n}, \bm{V}^{(A)}_{1:n} \leftarrow \bm{K}^{(B)}_{1:n}, \bm{V}^{(B)}_{1:n}
$$

按所有层 / 所有 head 同步执行；$n = n_{\text{text}} + k$，需要覆盖文本 token 段 + 早期 acoustic（或语义） token buffer $k$ 才能真正生效（[[StyleSelfReferencing]] §4.4 / Fig.6 实证 $k=0$ 时反向）。

## 核心要点

1. **本质**：利用 KV cache 是"decoder 对历史 token 的内部表示"这个事实，强行换掉这部分内部表示。
2. **不动模型权重**，纯推理介入，属于**训练-free** 控制手段。
3. **必须配合 [[Sliding-Window Attention]] mask** 一起用——否则 self-attention 仍能看到中间段（$n+1..t^*$）的旧风格 token，效果被稀释。
4. **典型场景**：句内风格切换 / 上下文中途改写 / 多 prompt 拼接合成。

## 代表工作

- [[StyleSelfReferencing]] (Kang et al. 2026, arXiv:2605.27376): 首次在 prompt-based TTS 中提出 + 用 Decoder-A/B 双 decoder 设置实现 + 在 [[Parler-TTS]]-mini 上做训练-free 句内风格切换

## 与相邻技术的关系

- 与传统 [[KV Cache]] 优化（FlashAttention / PagedAttention）**正交**——它们关注效率，KV-Cache Swap 关注功能（控制）。
- 与 [[Attention Sink]] 的洞察互补——后者发现"早期 token 是 anchor"，KV-Cache Swap 是"主动换 anchor"。
- 与 prompt-tuning / activation steering 同属 inference-time intervention 大类，但介入层不同：activation steering 改 hidden activation；KV-Cache Swap 改 self-attention 的 K/V 历史。

## 相关概念

- [[KV Cache]]
- [[Sliding-Window Attention]]
- [[Attention Sink]]
- [[Self-Attention]]
- [[Style Self-Referencing]]
- [[StyleSelfReferencing]]
