---
type: concept
aliases: [R_struc, Sentence-Level Reward, 句子级重构奖励]
---

# Sentence-Level Reconstruction Reward

## 定义

[[DG-WGPO]] 中针对**高 [[WER]] 区段**的动态奖励项。当 token 级对齐已经失效（整段崩坏），改用最长公共子序列（LCS）占比 + 长度一致性的平均，鼓励整体结构对齐。

## 数学形式

$$
R_{struc} = \frac{1}{2}\cdot\frac{\text{LCS}(H, R)}{|R|} + \frac{1}{2}\cdot\max\left(0,\ 1 - \frac{\big||H|-|R|\big|}{|R|}\right)
$$

- $\text{LCS}(H, R)$：hypothesis 与 reference 的最长公共子序列长度
- $|H|, |R|$：分别是 hypothesis 和 reference 的 token 长度

## 核心要点

1. 用于 [[WER]] ≥ $\tau$（论文 $\tau$=0.3）区段，错误集中在句子级（整段 [[ASR Hallucination|幻觉]] / 漏识）。
2. LCS 项：奖励"关键骨架对上"。
3. 长度项：抑制过短（漏识）和过长（幻觉填充）。
4. 消融显示去掉 $R_{struc}$ 影响最大（7.35 → 7.54），是 [[DG-WGPO]] 最关键的组件。

## 代表工作

- [[MegaASR]] / [[DG-WGPO]]

## 相关概念

- [[DG-WGPO]]
- [[Token-Level Refinement Reward]]
- [[WER-Gated Fusion]]
- [[ASR Hallucination]]
