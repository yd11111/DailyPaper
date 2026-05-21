---
type: concept
aliases: [Causal LSTM, Unidirectional LSTM, 因果 LSTM]
---

# Causal LSTM

## 定义

单向 LSTM（左到右、严格按时间顺序处理），常用于**因果探针**（causal probing）场景 —— 时刻 $t$ 的输出只能依赖 $\leq t$ 的输入，不允许任何未来信息泄漏。是对内部表示是否**提前编码**某些信号做严格验证的工具。

## 数学形式

给定输入序列 $\mathbf{x}_1, \mathbf{x}_2, \ldots, \mathbf{x}_T$，LSTM 在每个时刻输出 $\mathbf{h}_t = \mathrm{LSTM}(\mathbf{x}_t, \mathbf{h}_{t-1})$。在探针场景下常与**强制延迟 $\delta$** 结合：

$$
\hat{y}_t = \mathbf{W} \cdot \mathrm{LSTM}(\mathbf{z}_{t-\delta}) + \mathbf{b}
$$

让探针只能看 $t-\delta$ 时刻及之前的特征 $\mathbf{z}$ 来预测 $t$ 时刻的目标，从而把"现在已经知道未来"和"现在能从过去预测未来"明确分开。

## 核心要点
1. **因果约束的关键作用**：在 turn-taking / EOI 等任务上，双向 RNN 会作弊（窥探到未来）；因果 LSTM 给出更可信的 AUC
2. **延迟扫描的科学性**：$\delta$ 从 0 扫到长延迟，AUC 仍高 → 内部表示**提前**编码了目标信号
3. **轻量易训**：H=64 这种小模型就够做探针，避免探针本身记忆 spurious 特征
4. **替代方案**：因果 Transformer、Linear Probe（无时序）、Logistic Regression（最严格）

## 代表工作
- [[Synchronization-Turn-Taking]]: 用 H=64 的因果 LSTM + 强制延迟 $\delta \in [0, 1920]$ ms 探针 [[Moshi]] hidden state，测 [[EOI Prediction]] 与 [[Hold vs Non-Hold]] 的可解码性

## 相关概念
- [[Probing]]
- [[CKA]]
- [[EOI Prediction]]
- [[Hold vs Non-Hold]]
