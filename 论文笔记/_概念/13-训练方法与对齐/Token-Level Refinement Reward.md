---
type: concept
aliases: [R_fine, Token-Level Reward, Token 级精修奖励]
---

# Token-Level Refinement Reward

## 定义

[[DG-WGPO]] 中针对**低 [[WER]] 区段**的动态奖励项，按 token 级编辑距离细分 hard / soft 替换：正确 token 计分最高，soft 替换（编辑距离相似度 ≥ 0.5）打折，hard 替换/插入/删除重惩。

## 数学形式

**Token 相似度**：

$$
\text{sim}(h, r) = 1 - \frac{\text{edit}(h, r)}{\max(|h|, |r|)} \in [0, 1]
$$

**精修奖励**：

$$
R_{fine} = \frac{n_C}{n_C + n_{hard} + \alpha_s \cdot n_{soft} + \epsilon}
$$

- $n_C$：正确 token 数
- $n_{hard}, n_{soft}$：hard / soft 错 token 数
- $\alpha_s$：soft 错折扣系数（论文取 0.4）
- $\epsilon = 10^{-8}$：数值稳定项

## 核心要点

1. 用于 [[WER]] < $\tau$（论文 $\tau$=0.3）区段，错误集中在 token 级时。
2. 区分 hard / soft 替换，让模型在易混词上拿到部分分数，加快学习。
3. 与 [[Sentence-Level Reconstruction Reward]] 通过 [[WER-Gated Fusion]] 加权融合。

## 代表工作

- [[MegaASR]] / [[DG-WGPO]]

## 相关概念

- [[DG-WGPO]]
- [[Sentence-Level Reconstruction Reward]]
- [[WER-Gated Fusion]]
