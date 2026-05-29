---
type: concept
aliases: [单调对齐, MA, Phoneme Monotonic Alignment]
domain: TTS
tags: [monotonic-alignment, robustness, decoder-only-tts, alignment]
created: 2026-05-29
last_updated: 2026-05-29
related_maps:
  - "[[TTS-核心挑战]]"
---

# Monotonic Alignment（单调对齐）

## 定义

一种约束 TTS 中文本（phoneme）与声学序列对齐关系的机制，强制对齐满足**单调性**——文本指针随生成只能"留在当前"或"前进一个"，不可回退或跳跃。用于消除自回归 TTS 的漏读、重复、错读和注意力坍塌。

> 与 [[Monotonic Alignment Search]]（MAS，Glow-TTS 训练期用 Viterbi 搜对齐）区分：本词条特指 **推理期** 的单调对齐策略（如 [[VALL-E R]]），把单调约束迁移到 decoder-only Transformer 的解码过程。

## 数学形式

推理第 $i$ 步，当前 phoneme 为 $p^t_j$，模型给出保持/前进的相对概率 $e_{i,j}$，由 Bernoulli 采样决定指针是否前进：

$$
z_{i,j} \sim Bernoulli\!\left(\frac{1}{1+\exp(e_{i,j})}\right)
$$

## 核心要点

1. **三性质**（Tan et al. 2021）：
   - **Locality**：每个 phoneme 对应一或多个连续声学 token，每个声学 token 唯一归属一个 phoneme → 防错读
   - **Monotonicity**：phoneme 顺序 = 声学 token 顺序 → 防重复
   - **Completeness**：每个 phoneme 至少 1 个声学 token → 防漏字
2. 经典单调注意力只适用于 encoder-decoder 结构；推理期 MA 把它迁移到 decoder-only，训练几乎零额外负担。
3. 训练阶段通常需要外部强制对齐（如 [[MFA]]）提供逐帧 phoneme 标签。

## 代表工作

- [[VALL-E R]]: 把 phoneme 预测整合进训练，在推理期用 MA 约束生成，WER 逼近 ground truth

## 相关概念

- [[Monotonic Alignment Search]]
- [[Forced Alignment]] / [[MFA]]
- [[Duration Predictor]]
