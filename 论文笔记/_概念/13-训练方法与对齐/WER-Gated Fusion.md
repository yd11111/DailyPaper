---
type: concept
aliases: [WER 门控融合, R_dynamic gating, WER-Gated Reward Fusion]
---

# WER-Gated Fusion

## 定义

[[DG-WGPO]] 的核心机制：根据当前样本 [[WER]] 与阈值 $\tau$ 在 [[Token-Level Refinement Reward|$R_{fine}$]]（token 级精修）和 [[Sentence-Level Reconstruction Reward|$R_{struc}$]]（句子级重构）之间动态切换融合权重。

## 数学形式

$$
R_{dynamic} = \begin{cases} 0.75\, R_{fine} + 0.25\, R_{struc}, & \text{WER} < \tau \\ 0.25\, R_{fine} + 0.75\, R_{struc}, & \text{WER} \geq \tau \end{cases}
$$

论文取 $\tau = 0.3$。

## 核心要点

1. 设计依据：低 [[WER]] 错误集中在 token 级，高 WER 错误集中在句子级。
2. 阈值敏感性平坦（Table 9：$\tau \in [0.2, 0.5]$ 都接近 7.64 的 [[NOIZEUS]] WER），鲁棒。
3. 消融显示去掉 gated fusion（改为等权融合）后 [[VOiCES]] 从 7.35 → 7.41，证明门控本身有效。

## 代表工作

- [[MegaASR]] / [[DG-WGPO]]

## 相关概念

- [[DG-WGPO]]
- [[Token-Level Refinement Reward]]
- [[Sentence-Level Reconstruction Reward]]
- [[WER]]
