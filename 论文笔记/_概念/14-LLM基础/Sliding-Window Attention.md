---
type: concept
aliases: [Sliding-Window Attention, Sliding Window Attention, sliding-window attention, SWA, Local Attention]
domain: LLM
tags: [attention, transformer, long-context, inference-optimization]
created: 2026-05-29
last_updated: 2026-05-29
---

# Sliding-Window Attention

## 定义

对 self-attention 加 mask，使每个 query token 只能 attend 到**最近 $w$ 个 token**（局部窗口），把全局二次复杂度降到 $O(T \cdot w)$。常与 **global anchor token**（如 [[Attention Sink]] 的初始几个 token）配合使用，避免长序列中早期信息完全丢失。

## 数学形式

基本滑窗 mask：

$$
M_{ij} = \begin{cases} 0 & \text{if } i - w \leq j \leq i \\ -\infty & \text{otherwise} \end{cases}
$$

带 global anchor 的扩展版本（如 [[StyleSelfReferencing]] §4.2 Eq.4）：

$$
M_{ij} = \begin{cases} 0 & \text{if } j \leq n \,\, \text{or} \,\, i - w \leq j \leq i \\ -\infty & \text{otherwise} \end{cases}
$$

前 $n$ 个 anchor 位置始终可见 + 最近 $w$ 个本地 token 可见。

## 核心要点

1. **来源**：Longformer (Beltagy et al. 2020) 提出 → Mistral / Mixtral / Gemma 等现代 LLM 广泛采用。
2. **典型 $w$**：1k-8k tokens（视模型而定）。
3. **必须配合 anchor**：纯滑窗在长序列上会丢失早期上下文，导致输出漂移 → [[Attention Sink]] (Xiao et al. 2024) 揭示保留前几个 token 作为 attention sink 能稳定下来。
4. **作为推理时介入手段**：除训练时使用，也可在推理时**临时加入**做受控生成（[[StyleSelfReferencing]] 是案例：在 $t^*$ 之后才施加滑窗 mask，把中间段的源风格 token 屏蔽，让 decoder 只看最新 token + 替换后的 anchor）。

## 在音频领域的应用

- 流式 ASR / TTS 的低延迟生成（限制 attention 范围 → 低首包延迟）
- [[StyleSelfReferencing]] (2026) 把它和 [[KV-Cache Swap]] 配合做句内风格切换
- Long-form audio 处理（>1 分钟音频）

## 局限

- 窗口外的全局依赖完全丢失（除非有 anchor）
- 在某些任务（需要长依赖的指代消解 / 跨段一致性）上性能下降

## 代表工作

- Beltagy et al. (2020) Longformer
- Xiao et al. (2024) StreamingLLM with [[Attention Sink]]
- [[StyleSelfReferencing]] (2026): 在 TTS 推理时临时启用做控制

## 相关概念

- [[Self-Attention]]
- [[Attention Sink]]
- [[KV Cache]]
- [[KV-Cache Swap]]
