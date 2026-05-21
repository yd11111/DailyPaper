---
type: concept
aliases: [抗复读奖励, R_rep, Repetition Penalty Reward]
---

# Anti-Repetition Reward

## 定义

RL 微调 ASR / [[Speech LLM]] 时使用的**硬门控奖励**：若模型输出中重复 n-gram 超过阈值，整条 rollout 奖励直接判 0；否则为 1。专门抑制 RL 训练中常见的"复读机式" [[ASR Hallucination|幻觉]]。

## 数学形式

$$
R_{rep}(H) = \begin{cases} 0, & \text{if repeated n-grams above threshold} \\ 1, & \text{otherwise} \end{cases}
$$

通常与其他奖励项相乘组合：

$$
R_{static} = R_{rep} \cdot R_{wer}
$$

## 核心要点

1. 是 0/1 硬门控，比 token-level 软惩罚更有效——可一票否决。
2. 解决的核心病：RL 优化 [[WER]] 时模型会发现"重复正确词"也能拿分，于是退化成复读机。
3. [[MegaASR]] 消融显示去掉 $R_{rep}$ 后 [[VOiCES]] [[WER]] 从 7.35 → 7.46。

## 代表工作

- [[MegaASR]] / [[DG-WGPO]]

## 相关概念

- [[DG-WGPO]]
- [[ASR Hallucination]]
- [[DAPO]]
