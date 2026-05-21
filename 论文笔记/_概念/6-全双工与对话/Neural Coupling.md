---
type: concept
aliases: [Neural Coupling, Speaker-Listener Coupling, 神经耦合]
---

# Neural Coupling

## 定义

神经科学中的现象：在对话过程中，**说话者与听话者的脑活动会在时间和空间上对齐**（time-locked、region-aligned）。fMRI / EEG 研究还发现**听者大脑会提前预测说者**的内容；这种提前耦合（anticipatory coupling）越强，理解力越好。是 [[Entrainment]] 在神经层面的对应。

## 核心要点
1. **测量方式**：常用脑区间 BOLD 信号的时间相关、phase-locking、滞后相关、信息论度量
2. **anticipatory 成分**：听者活动有时**领先**说者活动（即负 lag 处出现峰值），被解释为预测
3. **与理解力正相关**：耦合强度可预测听者理解程度
4. **机器对话类比**：可把 LLM/SDM 的 hidden state 当作"神经活动"，用 [[CKA]] / [[Mutual Information]] 测耦合

## 代表工作
- Stephens, Silbert & Hasson 2010, "Speaker–listener neural coupling underlies successful communication"（PNAS）—— 经典 fMRI 研究
- Hasson 实验室一系列后续工作
- [[Synchronization-Turn-Taking]]: 把这套思路搬到 [[FDSDS]]，用两个 [[Moshi]] 间的 [[CKA]]-vs-lag 曲线类比人类神经耦合

## 相关概念
- [[Entrainment]]
- [[CKA]]
- [[FDSDS]]
