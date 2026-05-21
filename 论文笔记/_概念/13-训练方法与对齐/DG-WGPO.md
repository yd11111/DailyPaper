---
type: concept
aliases: [Dual-Granularity WER-Gated Policy Optimization, DG WGPO]
---

# DG-WGPO

## 定义

**Dual-Granularity WER-Gated Policy Optimization**：[[MegaASR]] 在 [[DAPO]] 基础上提出的鲁棒 ASR 强化学习目标。核心：根据当前样本 [[WER]] 与阈值 $\tau$ 在 **token 级精修奖励** $R_{fine}$ 与 **句子级重构奖励** $R_{struc}$ 之间门控融合。

## 数学形式

**WER 门控动态奖励**：

$$
R_{dynamic} = \begin{cases} 0.75\, R_{fine} + 0.25\, R_{struc}, & \text{WER} < \tau \\ 0.25\, R_{fine} + 0.75\, R_{struc}, & \text{WER} \geq \tau \end{cases}
$$

**最终奖励**：

$$
R = (1-\alpha_{dyn})\, R_{simple} + \alpha_{dyn}\, R_{dynamic}
$$

其中 $R_{simple} = R_{rep} \cdot R_{wer}$ 是静态奖励层。论文取 $\tau = 0.3$、$\alpha_{dyn} = 0.6$。

## 核心要点

1. 观察：低 WER 错误集中在 token 级（替换/删除），高 WER 错误集中在句子级（整段幻觉/漏识）。
2. 双粒度奖励：
   - $R_{fine}$（[[Token-Level Refinement Reward]]）：区分 hard/soft 替换，按编辑距离相似度判断
   - $R_{struc}$（[[Sentence-Level Reconstruction Reward]]）：LCS 占比 + 长度一致性
3. 抗复读硬门控 $R_{rep}$（[[Anti-Repetition Reward]]）杜绝 RL 训练中的"复读机式" [[ASR Hallucination|幻觉]]。
4. 消融显示去掉 $R_{struc}$ 影响最大（7.35 → 7.54）。

## 代表工作

- [[MegaASR]]：原始提出

## 相关概念

- [[DAPO]]
- [[WGPO]]
- [[GRPO]]
- [[WER-Gated Fusion]]
- [[Token-Level Refinement Reward]]
- [[Sentence-Level Reconstruction Reward]]
- [[Anti-Repetition Reward]]
- [[A2S-SFT]]
