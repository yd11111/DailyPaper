---
type: concept
aliases: [Curriculum Learning, 课程学习]
---

# Curriculum Learning

## 定义
逐步增加训练任务难度的训练范式，让模型从简单样例开始学起再过渡到复杂样例。

## 核心要点
1. 由 Bengio 2009 提出
2. 音频对话场景：modality alignment → half-duplex → full-duplex
3. 比直接训练 full-duplex 更稳定，效果显著更好

## 代表工作
- [[OmniFlatten]] 三阶段训练完美演示该思路；Table 3 ablation 证明每加一阶段 chat 分数都涨

## 相关概念
- [[OmniFlatten]]
