---
type: concept
aliases: [置信度引导迭代解码, Confidence-guided Refinement, Confidence-based Sampling]
domain: TTS
tags: [confidence-decoding, non-autoregressive, refinement, maskgit]
created: 2026-05-29
last_updated: 2026-05-29
related_maps:
  - "[[TTS-技术路线图]]"
---

# Confidence-based Iterative Decoding

## 定义
[[Masked Generative Transformer|掩码生成模型]]的迭代解码策略：每轮按预测置信度（通常取预测分布最大概率）对低置信 token 重掩码并重预测，逐轮提质。源自 MaskGIT，常用作 NAR 生成与精修阶段。

## 数学形式
置信度定义为预测分布最大概率的对数（负 min-entropy）：

$$
\mathbf{C}_{(t)} = \log P^{max}_{(t)}
$$

对最低 $\gamma$ 分位 token 重掩码重预测；已更新 token 置信度可永久置 1 防重复精修（[[PALLE]] Stage 2 做法）。

## 核心要点
1. 用置信度排序决定每轮"信哪些、改哪些"。
2. 既可用于从零生成（MaskGIT），也可用于对已有草稿做精修（[[PALLE]] Stage 2）。
3. 步数少（[[PALLE]] 约 7 步即收敛）。

## 代表工作
- [[PALLE]]: Stage 2 用置信度引导精修 Stage 1 的 PAR 草稿，把 SIM-o 由 ~0.712 提到 ~0.717。
- [[MaskGCT]]: 用置信度调度做 NAR 生成。

## 评测/常见数字
[[PALLE]]：Stage 2 约 7 步，cross-sentence WER-W 由 2.58（stage-one-only）降到 2.23。

## 相关概念
- [[Masked Generative Transformer]]
- [[Non-Autoregressive Model]]
- [[Pseudo-Autoregressive]]
