---
type: concept
aliases: [Style Self-Referencing, style self-referencing, 风格自参照]
domain: TTS
tags: [tts, controllable-tts, attention, ar-decoder, set-and-maintain, inference-time-control]
created: 2026-05-29
last_updated: 2026-05-29
based_on:
  - "Kang-2026-arXiv:2605.27376"
related_maps:
  - "[[TTS-核心挑战]]"
  - "[[TTS-技术路线图]]"
---

# Style Self-Referencing

## 定义

Kang et al. (2026) 在 [[StyleSelfReferencing]] 论文中首次命名的 prompt-based AR TTS decoder 中的现象：**风格信息在生成早期被读入 acoustic token，之后 cross-attention 进入"set-and-maintain"模式，对后续 cross-attn 的 prompt 替换不再敏感**——decoder 通过 self-attention 自参照早期已编码风格的 acoustic token 维持一致性。

## 核心机制

1. **早期生成阶段**：cross-attention 在各 style token 上活跃更新，attention 分布波动大（Eq.5 量化方差高）。
2. **后期生成阶段**：cross-attention 权重固化，并把权重大量分给信息量低的词（"with", `<EOS>`）；attention 方差接近 0。
3. **直接后果**：在 $t^*$ 中途换 prompt 不奏效——decoder 已不再"听" prompt，而是参照自己早期生成的 token。

## 数学量化

$$
\text{Var}(t) = \frac{1}{|S|} \sum_{s \in S} \left(a_{t,s} - \bar{a}_t\right)^2
$$

早期 $t$ 上 $\text{Var}(t)$ 高，后期 $\text{Var}(t) \to 0$。

## 与已知现象的关系

- 与 [[Attention Sink]]（Xiao et al. 2024, "streaming language models with attention sinks"）**同源**——都是 AR decoder 早期 token 主导后续生成的通用模式。
- Style Self-Referencing 是该通用模式在 **prompt-based TTS 风格控制场景**的具体表现 + 量化诊断 + 对应缓解方案的命名。

## 缓解方案

由提出论文同步给出（详见 [[StyleSelfReferencing]]）：
- [[KV-Cache Swap]]：在过渡点把前 $n$ 个位置的 self-attention K/V 替换为目标风格 prompt 单独跑出来的 K/V。
- [[Sliding-Window Attention]]：在 $t^*$ 之后只保留替换后的初始 anchor + 最近 $w$ 个 token，把中间段（带源风格）屏蔽。
- 单独换 cross-attn 的 style embedding 无效（[[StyleSelfReferencing]] Table 4 消融）。

## 代表工作

- [[StyleSelfReferencing]] (Kang et al. 2026, arXiv:2605.27376): 首次命名 + 量化 + 在 [[Parler-TTS]]-mini 上验证 + 给出缓解方案

## 未验证的开放问题

- 是否在 [[VALL-E]] / [[CosyVoice]] / [[InstructTTS]] 等其他 prompt-based / codec-LM AR TTS 上同样存在？强度如何？
- 在 NAR TTS（[[FastSpeech]] / [[FastSpeech 2]]）上是否完全不存在？
- 与 [[Attention Sink]] 在 LM streaming 的关系：是否所有 AR 序列模型都有"early-token 主导"通病？

## 相关概念

- [[Attention Sink]]
- [[Cross-Attention]]
- [[Self-Attention]]
- [[KV-Cache Swap]]
- [[Sliding-Window Attention]]
