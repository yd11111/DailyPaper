---
type: concept
aliases: [Probing, Probe, Diagnostic Classifier, 探针]
---

# Probing

## 定义

在预训练模型的 hidden state 上**外挂一个轻量分类器**，测试模型内部表示中是否**已经编码**了某种语言/声学/任务相关信息。Probe 通常很小（线性层 / 单层 MLP / 小 LSTM），目的是限制 probe 自身的学习能力，让结果反映"表示已编码"而非"probe 重新学到"。

## 数学形式

给定冻结的模型 $\mathcal{M}$ 输出表示 $\mathbf{h}$ 与目标标签 $y$，训练 probe $f_\theta$：

$$
\min_\theta\ \mathbb{E}\!\left[\mathcal{L}\big(y,\, f_\theta(\mathbf{h})\big)\right]
$$

主模型参数冻结，只更新 $\theta$。

**因果 probe**：进一步加上延迟 $\delta$ 与因果架构（如 [[Causal LSTM]]），只允许 probe 用 $\mathbf{h}_{t-\delta}$ 来预测 $t$ 时刻目标，从而严格分离"未来信息泄漏"与"提前编码"。

## 核心要点
1. **probe 容量是关键超参**：容量过大会自己学到信号，结果不能解释模型内部
2. **shuffled-label baseline**：把标签洗乱再训一遍，估计 probe 的 floor performance
3. **跨层比较**：在不同层做相同 probe，可绘制"信息深度"曲线，定位某能力的涌现层
4. **任务设计驱动结论**：probe 任务的选择决定能问什么问题（语法 vs 语义 vs 时序）

## 代表工作
- 大量 NLP 工作（如 Hewitt & Manning 2019 "A Structural Probe"）
- [[Synchronization-Turn-Taking]]: 在 [[Moshi]] hidden state 上用 [[Causal LSTM]] 做 [[EOI Prediction]] / [[Hold vs Non-Hold]] 的因果探针

## 相关概念
- [[Causal LSTM]]
- [[CKA]]
- [[Mutual Information]]
